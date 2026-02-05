
----
 - Tags:  #HTB #Easy #linux #SSRF #OWASPTOP10 #web 
- --- 
#### QUÉ SE EXPLOTA?

Virtual Hosting
Abusing File Upload
Server Side Request Forgery (SSRF) Exploitation + Internal Port Discovery
API enumeration through SSRF
Private Github Project Enumeration + Information Leakage
Abusing sudoers [Privilege Escalation] - GitPython Exploitation (CVE-2022-24439)

#### Write Up

##### PRIMERA FASE: Reconocimiento

Vamos a empear la maquina **haciendo un reconocimiento a través de ICMP** esto nos va a permitir determinar el **TTL** que nos dirá si el **host está activo** y además nos va a permitir **determinar que SO se está ocupando**. 

![](../imgs/Pasted image 20251118083239.png)

**Posteriormente vamos a hacer nuestros 2 escaneos**. El primer escaneo a realizar **se trata de dterminar todos los puertos abiertos** además hay que recordar que para el primer escaneo ocupamos un **SYN SCAN**

![](../imgs/Pasted image 20251118083257.png)

Posteriormente **con los scripts de nmap realizaremos un escaneo a mayor detalle** esto para reconocer de mejor manera ante lo que estamos.

![](../imgs/Pasted image 20251118083313.png)

Vemos que **nmap** nos está redirigiendo al dominio **editorial.htb** por lo que proseguimos a añadirlo de forma manual al **[[etc-hosts]]**. 

![](../imgs/Pasted image 20251118085531.png)

Proseguimos a inspeccionar la página web corriendo. 

![](../imgs/Pasted image 20251118085624.png)

No encontramos nada realmente relevante. **Lo que encontramos como posible vector** es la parte de subida de archivos. **En la sección *Publish with us*** podemos interactuar con la página. 

![](../imgs/Pasted image 20251118085824.png)

**Con [[Burpsuite]]** vamos a analizar la respuesta de la página al subir archivos para ver mayores detalles.. 

**Intentamos subiendo un archivo malicioso .php** pero, al subir el archivo, no nos devolvía nada. 
```php
system($GET_['cmd']);
```

![](../imgs/Pasted image 20251118102716.png)
**Lo volvimos a subir y en el boton de preview** nos devolvía respuesta, sin embargo, únicamente nos proporcionaba un link, que más allá de permitir ejecutar comandos, nos descargaba el .php que acabamos de subir. 

![](../imgs/Pasted image 20251118102745.png)

##### SEGUNDA FASE: Explotación

Nos percatanmos que necesitamos pasarle una **url** para que cargue la cover desde algún recurso externo. **Nos montamos un servidor** para ver si podemos entablar una conexión hacia nosotros. 
![](../imgs/Pasted image 20251118215447.png)
**La página hace peticiones hacia afuera** aunque de todos modos no nos serviría de nada, ya que anque pasaramos algún **.php** lo más seguro es que nos lo descargaría. **Eso nos lleva a pensar en un [[SSRF]]**. **Ya que tenemos control sobre la url** podemos apuntar hacía la misma maquina para que nos muestre que otros puertos o servicios internos tiene corriendo. 

**Guardamos la petición de [[Burpsuite]] y la modificamos para usarla con [[ffuf]]** limpiamos los headers y dejamos los **[[Headers necesarios]]**. 

![](../imgs/Pasted image 20251118223000.png)

```bash
ffuf -u [url] -request archivo.request -z range,0-65535 -ac
```

![](../imgs/Pasted image 20251118223911.png)

Encontramos que el puerto 5000 está abierto. 

![](../imgs/Pasted image 20251118224223.png)

**Hacemos la petición con burpsuite** y nos devuelve una ruta. Al poner la ruta en el buscador se nos descarga un archivo. 

**El archivo es un [[json]]** por lo tanto lo visualizamos con **jq**

![](../imgs/Pasted image 20251118224338.png)

**Encontramos varias rutas interesantes**... Ya que hay **[[SSRF]]** en **[[Burpsuite]]** probamos con:

![](../imgs/Pasted image 20251118224514.png)

**Se nos descarga un archivo y al abrirlo**... había claves. 

![](../imgs/Pasted image 20251118224948.png)

Nos conectamos a través de **[[SSH]]** y listo **tenemos accesso al servidor**. 

![](../imgs/Pasted image 20251118225120.png)

##### TERCERA FASE: PostExplotación - Privesc

Una vez que ya estamos como el usuario **dev** empezamos a investigar **posibles vectores para escalar privilegios**. Lo primero vemos que en el directorio en el que estamos esta la app

![](../imgs/Pasted image 20251119145135.png)

**Descubrimos que dentro de /apps, hay un repositorio .git** 

![](../imgs/Pasted image 20251119145204.png)

Ahora proseguimos a **investigar si hay commits o logs para revisar**. 
![](../imgs/Pasted image 20251119145249.png)

Los revisamos y encontramos uno **conteniendo otras credenciales**. 

![](../imgs/Pasted image 20251119145343.png)

**Cambiamos de usuario a prod** 

Como el usuario PROD, podemos ejecutar **[[SUDO]]** 

![](../imgs/Pasted image 20251119145453.png)

Podemos ejecutar como root un script de python3 . Al investigar el Script vemos que utiliza la **[[libreria GitPython]]**.  **GitPython** tiene una vulnerabilidad que a través de un **transport** externo, es decir un protocolo externo, **permite ejecutar codigo**.

**Al ser un script perteneciente a root, se ejecuta como root** por tanto, si podemos ejecutar codigo como root, podríamos cambiar permisos a la **/bin/bash**.

Nos dirigimos al directorio */tmp* y creamos un **script que cambia permisos**.

![](../imgs/Pasted image 20251119154309.png)

Una vez hecho esto, le damos permisos de ejecución al script que cambia permisos, **ejecutamos el script privilegiado** como sudo apuntando al script que creamos, **de esa manera el script que cambia permiso a la bash se ejecutará como root** y podríamos obtener una bash con permiso SUID. 

```bash
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c /tmp/privesc'
```

![](../imgs/Pasted image 20251119163147.png)

**Y listo** 

658b777d27177a35ade971769f8a1149