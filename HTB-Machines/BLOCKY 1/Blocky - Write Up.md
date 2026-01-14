
--- 
- Tags: #HTB #SSH #wordpress
- --
#### QUÉ SE EXPLOTA?

WordPress Enumeration
Information Leakage
Analyzing a jar file - JD-Gui + SSH Access
Abusing Sudoers Privilege [Privilege Escalation]

#### Write UP

##### PRIMERA FASE: Escaneos

Cómo ya es costumbre iniciamos la **primer fase de reconocimiento** con los escaneos correspondientes. 
**El primer escaneo es para descubrir todos los puertos abiertos** 

**posteriormente** lanzamos otro escaneo para **obtener mayor información sobre los puertos previamente descubiertos**.

![[Pasted image 20251026131803.png]]

 **Vemos que hay 4 puertos abiertos** **21,22,80,25565** Además vemos que **La versión de [[ssh]] utilizada es vulnerable a enumeración de usuarios** en **[[searchsploit]] podemos encontrar un exploit para enumerar.** 

Además en el mismo escaneo de nmap vemos que al tratar de acceder al **puerto 80** nos redirige al nombre de un dominio por lo que seguramente lo tendremos que agregar al [[etc-hosts]]

**Ahora proseguimos a investigar el sitio web**. Para comenzar tiramos con **whatweb** un escaneo de reconocimiento web.

![[Pasted image 20251026133213.png]]
**Hay una versión de [[WordPress]] 4.8** lo cual seguramente nos lleva a encontrar vulnerabilidades. 

![[Pasted image 20251026133434.png]]

Encontramos la siguiente página... **Al estar hecha en wordpress** podemos ver **usuario** que creo el post si [[WordPress]] está mal configurado

![[Pasted image 20251026133559.png]]

Ya tenemos un **posible usuario** *NOTCH* }

![[Pasted image 20251026141647.png]]
##### SEGUNDA FASE: Reconocimiento Activo - Explotación

**Pasamos a seguir con la fase de reconocimiento áctivo** por lo cual ocuparemos [[GoBuster]] primeramente para descubrir posibles **subdominios y directorios** 

![[Pasted image 20251026135301.png]]

Obtuvimos las siguientes **rutas**, sin embargo, en la ruta **plugins** encontramos unos archivos **[[jar]]**, **para poder visualizar los archivos jar ocupamos [[jd-gui]]**

![[Pasted image 20251026141302.png]]
*archivos en rojo*

Al abrir [[jd-gui]] descubrimos que había **un usuario y contraseñas** disponibles, así que intentamos **reutilizar la contraseña junto con el usuario que descubrimos previamente**.

![[Pasted image 20251026141951.png]]

No funcionó en **wp-admin**, pero, recordemos que en [[ssh]]
podemos enumerar usuarios validos. 

![[Pasted image 20251026142010.png]]

**Ocupamos el linux/remote/45939.py**

![[Pasted image 20251026143943.png]]

**notch is a valid username**

![[Pasted image 20251026144036.png]]
##### TERCERA FASE: Post Explotación - Privesc

**Primero checamos [[SUDO]]** y vemos que nos pedía contraseña, intentamos con la que teníamos...

![[Pasted image 20251026144304.png]]

SOMOS **ROOT**!

