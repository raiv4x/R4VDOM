
--- 
- Tags: #windows #Easy #HTB #redes 
- -- 
#### QUÉ SE EXPLOTA?

Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow [RCE]
Token Kidnapping - Churrasco [Privilege Escalation]


#### WRITE UP 

**La primer fase de esta etapa como siempre empezamos haciendo los escaneos propios**. 

Primeramente determinamos el **SO** lanzando un paquete **ICMP** con **ping** para determinar ante que tipo de máquina estamos.  

![[Pasted image 20251120170545.png]]

El primer escaneo está enfocado en reconocer todos los puertos abiertos de un Dispositivo. **Para este escaneo utilizaremos un SYN scan** . 

Para la segunda parte del escaneo **utilizaremos los scripts que trae por defecto nmap**. El fin de esta segunda etapa es obtener más detalles de los puertos previamente descubiertos 

![[Pasted image 20251120170601.png]]

**Descubrimos el puerto 80 abierto** por lo que hay una página corriendo. **Además de eso, descubrimos diferentes [[Métodos HTTP]]** Y **[[WebDAV]]** disponible.

**En esta ocasión utilizamos [[davtest]] pero, no nos permitía usar ni un otro método** 

##### SEGUNDA FASE: Explotación

De acuerdo a nmap, el **server a utilizar es muy viejo** por lo que podría haber algún exploit. **Encontramos un CVE**

![[Pasted image 20251120171717.png]]

**[[CVE-2017-7269]]** Descargamos el exploit, lo ejecutamos con **python2** 

![[Pasted image 20251120172115.png]]

##### TERCERA FASE: Post Explotación - Privesc

**Empezamos con algunos [[Comandos utiles para escalar]]** en este caso en Windows

![[Pasted image 20251120135622.png]]

**Encontramos el [[SeImpersonatePrivilege]] encendido** por lo que podemos abusar de **[[JuicyPotato]] o [[churrasco.exe]]**. Generalmente cuando estamos **Ante una versión antigua como en este caso**:

![[Pasted image 20251120140445.png]]

**Utilizamos [[churrasco.exe]]** 

**Lo descargamos a través de [[SMB]]** y lo ponemos en el directorio **temp** 

```bash

impacket-smbserver smbFolder $(pwd) -smb2support
```


![[Pasted image 20251120143543.png]]

**Churrasco ejecuta comandos a nivel Authority system** por lo que si ejecutamos el **nc.exe** d enuestro recurso compartido y nos enviamos una shell **resiviremos la shell como AUhtority system**

**En esta ocasión** Nos ponemos en escucha con **[[NETCAT]]** nos copiamos su binario y através de **[[SMB]]** lo transferimos, por alguna razoń ganamos acceso aútomatico. 

![[Pasted image 20251120174221.png]]