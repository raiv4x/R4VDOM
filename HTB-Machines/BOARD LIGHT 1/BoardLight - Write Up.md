
--- 
- Tags: #HTB #bash #CVE #Easy 
- --
#### QUÉ SE EXPLOTA?

Subdomain Enumeration
Dolibarr 17.0.0 Exploitation - CVE-2023-30253
Information Leakage (User Pivoting)
Enlightenment SUID Binary Exploitation [Privilege Escalation]


#### Write UP

##### PRIMERA FASE: Escaneos - Reconocimiento

Como ya es costumbre vamos a **empezar la fase de reconocimiento escaneando todos los puertos disponibles de la maquina a tratar**. Para esta primer fase de escaneos **emplearemos un SYN SCAN**. 

Para la segunda fase **vamos a hacer un escaneo de los puertos previamente encontrados**. En esta fase el escaneo empleamos los scripts por defecto que ya trae nmap. Además **vamos a emplear un escaneo de versiones**. 

![[Pasted image 20251027081320.png]]
Encontramos el puerto **22** y **80** abiertos. Por lo que proseguimos a **inspeccionar la web con *whatweb***. 
![[Pasted image 20251027081742.png]]

**Encontramos un email, el cual nos revela un dominio**, tenemos que agregar el dominio al [[etc-hosts]]. 

**Antes de pasar a la fase activa de reconocimiento** vamos a ver la página web. 

![[Pasted image 20251027082436.png]]
**Es una página no funcional** entonces vamos a seguir practicamente con la fase de reconocimiento activo. 

##### SEGUNDA FASE: Reconocimiento Activo - Explotación  

**Empezaremos buscando directorios ocultos así como subdominios** esto lo estaremos haciendo con **[[wfuzz]]**, también podemos usar **[[GoBuster]]** para subdominios. 

![[Pasted image 20251027083826.png]]
![[Pasted image 20251027083901.png]]

**De rutas o directorios no encontramos nada** sin embargo **encontramos subdoiminios**. 

Agregamos el **subdominio** al [[etc-hosts]] y listo. 

![[Pasted image 20251027084236.png]]

Un panel de **login** para un **ERP/CRM** **[[Dolibarr]]**. Además de eso. Vemos que la versión de **Doli** es la **17.0.0.** 

**Proseguimos a intentar con  [[las credenciales genericas más usadas]]**. Y pudimos acceder con **admin:admin**. 

![[Pasted image 20251027085751.png]]

Una vez dentro **seguimos a investigar si hay alguna vulnerabilidad** para esto ocuparemos [[searchsploit]]. 

En **searchsploit** había unos cuantos exploits pero, no encontramos muchos relacionados a dolibarr 17.0. por lo que **proseguimos a buscar en google**. 
![[Pasted image 20251027090158.png]]

**El exploit que encontramos** requiere de acceso y además al parecer es una inyección de codigo **php**.  Checamos el script y básicamente la **vulnerabilidad** está en la sección de creación de website y edición del codigo del mismo..**[[CVE-2023-30253]]

![[Pasted image 20251027115612.png]]

al parecer la vulnerablididad esta en la pagina de edición, primero se tiene que crear un sitio y una pagina. **Posteriormente editar el html** y ahí está el vector. 

**descargamos el exploit y lo ejecutamos**

![[Pasted image 20251027120026.png]]

![[Pasted image 20251027120035.png]]

**Finalmente estabilizamos la shell como siempre** [[Establización de shell]]

##### TERCERA FASE: Post Explotación - Privesc

**Empezamos a buscar vectores de subida para subir privilegios** para eso ocupamos ayuda de los [[Comandos utiles para escalar]]. 

En este caso encontramos un binario que tenia privilegios SUID y el propietario era root:

![[Pasted image 20251027122041.png]]
**/usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys** 

**[[enlightenment]]**

**Checamos si tenía alguna vulnerabilidad** y efectivamente si tenía. 

![[Pasted image 20251027122058.png]]

**No tenemos la versión** por lo que la intentamos buscar pero, no nos permitía, al parecer es por que no tenemos permisos en algunas rutas. 

**Seguimos buscando posibles vectores para escalar** y encontramos un archivo **conf.php** que nos estaba dando una contraseña. **Guardamos esa contraseña y la tratamos de utilizar para él unico usuario que estaba registrado** 


![[Pasted image 20251027124829.png]]

![[Pasted image 20251027124903.png]]

**Cómo Larissa si pudimos ver la version de enlightenment** 

![[Pasted image 20251027124947.png]]

**Vemos que si es vulnerable**

![[Pasted image 20251027125039.png]]

**No funcionó el de searchsploit por lo que buscamos en la web y encontramos** 
https://github.com/d3ndr1t30x/CVE-2022-37706/blob/main/exploit.sh

Y LISTO

