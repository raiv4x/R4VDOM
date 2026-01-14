
--- 
- Tags: #linux #HTB #Medium #wordpress  #Esteganografía 
- --- 
#### QUÉ SE EXPLOTA?

Wordpress Enumeration
Image Stego Challenge - Steghide
Information Leakage - User Enumeration
WordPress Exploitation - Theme Editor [RCE]
Abusing misconfigured permissions [Privilege Escalation]

#### WRITE UP 

##### PRIMERA FASE: Reconocimiento

Como ya es costumbre, vamos a empezar lanzando un **ping** para poder determinar de primera el **sistema operativo y también para determinar si la maquina victima está activa** 

![[Pasted image 20251213194415.png]]

Estamos ante una maquina **Linux**

Empezamos la fase de escaneos **utilizando nmap** como ya sabemos el primer escaneo que vamos a realizar **tendrá por objetivo** encontrar todos los puertos abiertos y buscar posibles vectores de entrada. 
![[Pasted image 20251213194623.png]]

**Lo primero que encontramos es el puerto 80 y 22 abierto** por lo que ahora proseguimos al 2do escaneo. El segundo escaneo **tiene por objetivo obtener mayores detalles sobre los puertos previamente encontrados, por lo que ahora jusamos los scripts por defecto de nmap, junto con el de versión** 

![[Pasted image 20251213194753.png]]

Mientras el escaneo detallado se ejecuta, utilizaremos **whatweb para investigar la página web corriendo** ya que encontramos que el puerto **80** estaba abierto.  **Además el puerto 22 que corría ssh** tiene una versión vulnerable a enumeración. 

**Encontramos que estamos ante un wordpress** por lo que ya podríamos empezar a pensar en enumerar usuarios, **[[wpscan]]** entre otros. 

Pasamos a investigar la web. 

**Primeramente tenemos que agregar el dominio al [[etc-hosts]] ya que los recursos no cargaban** 

![[Pasted image 20251213195111.png]]

Una vez que ya habíamos hecho eso cargó bien

![[Pasted image 20251213195350.png]]

Inspeccionando el sitio, la barra de busqueda era funcional, además pudimos encontrar un **usuario llamado falaraki** 

Al no encontrar nada pasamos a **enumerar los directiorios** posibles con **[[wfuzz]]**, utilizamos wfuzz para poder aplicar mejores filtros y demás.  

primero aplicamos el filtro de codigo 404· 

![[Pasted image 20251214092112.png]]

```bash
--hc=404
```

Una vez que aplicamos el filtro. 

![[Pasted image 20251214092609.png]]

Ahora tenemos que **filtrar por el número de carácteres**. 

```bash
--hh=157
```

Ya con los filtros aplicados, no encontramos nada. **solo encontramos unas cuantas rutas de wordpress**. 

Proseguimos a crear un directorio personalizado a base de la misma página web. **Para esto utilizaremos [[cewl]]** . 

```bash

cewl http://apocalystic.htb -m 3 -w dicc.txt
```

Una vez tenien el diccionario, volvemos a hacer un enumeración con **[[wfuzz]]**. 

aplicamos los mismos filtros y ..

![[Pasted image 20251214124314.png]]

Encontramos una ruta.  Por lo que proseguimos a verificarla. 

SImplemente encontramos una página que tenía una imagen:

![[Pasted image 20251214125408.png]]

**Descargamos la imagen y con [[exiftool]]** verificamos los **[[Metadatos]]**. 
No encontramos nada de utilidad, así que probamos también con 

```bash
strings image.jpg
```

igual podríamos emplear un filtro para que únicamente extraiga palabras con minimos

```bash
-n 5

```


**Recordemos que strings extrae texto hardcodeado oculto en diferentes archivos**.  Tanto de binarios, imagenes, pdfs

No encontramos nada, pero, **aún nos falta explorar la posibilidad de que haya [[Esteganografía]].**
Ahora utilizando **[[steghide]]**

```bash 

steghide extract -sf image.jpg
```

![[Pasted image 20251214143350.png]]

Extraemos los datos, en este caso un **.txt** 

**Era una lista por lo que proseguimos a usarla como posible lista con alguna contraseña. 

##### SEGUNDA FASE: Explotación

Ya que teníamos una posible contraseña, intentamos hacer fuerza bruta con **[[wpscan]]**

```bash

wpscan --url http://apocalyst.htb --usernames falaraki --passwords list.txt
```

![[Pasted image 20251214145243.png]]

Encontramos que la contraseña es valida por lo que ahora simplemente tenemos que enviarnos una revshell.  Transclisiation

**Recrodemos que con [[WordPress]]** podemos ir a la sección de templates y modificar las 404 para enviarnos una **[[Reverse shell]]** 

Entramos y primeramente checamos el rol que tiene falaraki. **Encontramos que tenía rol de administrador** 

![[Pasted image 20251214145836.png]]

Nos fuimos al **editor y ahí mismo modificamos la plantilla del error 404** para así poder llamar al recurso y podernos entablar una conexión. 

**Para esto podemos utilizar la plantilla .php de pentest monkey** **.

![[Pasted image 20251214150912.png]]

modificamos la plantilla, y ahora con **con curl tenemos que ejecutar el recurso**, nos ponemos en escucha y ejectuamos el recurso. 

```bash

curl http://apocalyst.htb/?p=404.php
```

![[Pasted image 20251214151246.png]]

##### TERCERA FASE: Post-Explotación

Primeramente tenemos que estabilizar la shell.

Una vez habíendo hecho eso proseguimos a investigar posibles vectores de escalación. Primeramente **como www-data no podemos hacer mucho** toca investigar **binarios, cron, scripts, configs por contraseñas, incluso podríamos ver si la password encontrada igual nos sirve.** 

**Finalmente encontramos una falla** 

```bash

find / -type d -writable 2>/dev/null 

```

Dentro de los archivos a los cuales nosotros también teníamos acceso a modificar, **es decir, arcivhos writeables** estaba el **[[etc-passwd]]** por lo que pudimos escalar privilegios depositando una contraseña creada con **openssl**

```bash

openssl passwd
```

Modificamos la **:x: del user root en el [[etc-passwd]]**, por el **hash unix** creado con openssl y listo. 

![[Pasted image 20251214155932.png]]

**Tenemos root**

![[Pasted image 20251214160045.png]]

