
----
- Tags: #linux #Easy #HTB #SSH #CVE 
- --- 
#### QUÉ SE EXPLOTA?

Web Enumeration [[GoBuster]]
[[CrushFTP]] Exploitation 
[[Erlang OTP]] Exploitation and PrivEsc. 

#### Write Up

##### PRIMERA FASE: Reconocimiento

La primer fase como es de costumbre empezamos **analizando el TTL de la maquina** esto lo hacemos para **identificar el SO** al cual nos estamos enfrentando. 

Nos dió un **TTL de 63** Lo cual quiere decir que nos estamoso enfrentando ante una maquina Linux

![[Pasted image 20251123133408.png]]

Posteriormente proseguimos a realizar nuestros **2 escaneos de rutina**. El primer escaneo **tendría por objetivo descubrir todos los puertos abiertos de un dispositivo** además de emplear el **syn scan** para hacer un escaneo más rápido. 

Posteriormente con los puertos previamente definidos **procederemos a realizar un escaneo a mayor detalle de los escaneos previamente encontrados**

![[Pasted image 20251123171032.png]]

**Encontramos el puerto 80 abierto y el 22** , además automaticamente nos redirige a un sitio que tenemos que agregar de manera manual a **[[etc-hosts]]**

Inspeccionamos la web, y nos encontramos con lo siguiente:

![[Pasted image 20251123171217.png]]

También había una panel de registro...
**Investigamos a profundidad y no encontramos nada** por lo que pasamos a realizar un **[[Fuzzing]] de directorios**, encontramos un directorio, pero, no teníamos acceso.

**Proseguimos a  hacer un fuzzing de subdominios**. Al realizar el **Fuzzing** encontramos una ruta que nos redirigé a:

![[Pasted image 20251123171550.png]]

**Nos encontramos ante un [[CrushFTP]]** 

![[Pasted image 20251123171830.png]]

Buscamos diferentes VUlns relacionadas a **[[CrushFTP]]** y nos encontramos con: **[[CVE-2025-31161]]** **lo único fue buscar un exploit y listo**.

##### SEGUNDA FASE: Explotación

Ejecutamos el exploit y efectivamente funciona, nos crea una cuenta de administrador.

![[Pasted image 20251124113714.png]]

![[Pasted image 20251124113733.png]]

Una vez dentro **nos pusimos a explorar posibles vectores** finalmente encontramos que podíamos incluirnos **diferentes rutas del sistema**. 

**Encontramos que teniamos acceso a la ruta del dominio principal** 

![[Pasted image 20251124113917.png]]

Únicamente nos la compartimos y le dimos permisos de escritura, de esa manera **podríamos subir un archivo .php en caso de que sea posible** y entrablar una **[[Reverse shell]]** con la shell de **pentestmonkey** 

![[Pasted image 20251124114118.png]]

![[Pasted image 20251124114159.png]]

**Nos ponemos en escucha, lo llamamos a través de la url** 

En teoría al poder escribir archivos en el dominio principal, desde ahí mismo podríamos ejecutar el script para lanzarnos la **[[Reverse shell]]**

![[Pasted image 20251124114407.png]]


![[Pasted image 20251124114446.png]]

##### TERCERA FASE: Post-explotación - Privesc

**Lo primero que hacemos como regla, estabilizar la shell** en este caso no nos complicamos y utilizamos bash

```bash

script /dev/null -c bash
```

empezamos a buscar vectores sin embargo no encontramos nada claro por lo que proseguimos a usar **pspy64**.

Desde nuestra maquina **nos pasamos pspy64** con wget a la ruta **/tmp de la victima** 

![[Pasted image 20251124115516.png]]

**pspy** nos dio información interesante de un script que se estaba ejecutando. 

![[Pasted image 20251124120320.png]]

**Al parecer había un script del servicio [[Erlang OTP]]** corriendo.  Este servicio de [[Erlang OTP]] tenía el servicio **ssh** corriendo el cual corría como root. 

Investigamos el script...

![[Pasted image 20251124122412.png]]

**Encontramos no solamente el puerto por el cual corre el demonio de erlang** también encontramos un usuario y una contraseña. 

**Ya que el daemon** de erlang está corriendo en el puerto 2222 de acuerdo al script de configuración. Podemos conectarnos a él a través de **SSH**. 

![[Pasted image 20251124122717.png]]

**De acuerdo a [[Erlang OTP]]** se pueden ejecutar comandos del sistema.  

```
os:cmd("ls -la").
```

![[Pasted image 20251124122914.png]]

ya que está corriendo como root...
**Le damos permisos a la /bin/bash** y listo. 

![[Pasted image 20251124123016.png]]
**Ejecutamos la bash con permisos**

```bash
/bin/bash -p
```


![[Pasted image 20251124123034.png]]




