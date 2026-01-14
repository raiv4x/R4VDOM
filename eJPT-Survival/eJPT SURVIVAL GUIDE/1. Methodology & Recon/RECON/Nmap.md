
---
- Tags: #Recon #Herramienta #Firewall #WAFS #Metasploit
--- 

## 1. Descubrimiento de Hosts (Host Discovery)

*Se realiza al inicio cuando no tenemos idea de que dispositivos están encendidos*

| COMANDO               | ESCENARIO                                                                                      |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| `nmap -sn <red/CIDR>` | Ping Sweep: Escaneo rápido para ver qué IPs responden. No escanea puertos.                     |
| `nmap -Pn <IP>`       | Sin Ping: Úsalo si el host está encendido pero bloquea el ICMP. *Casi siempre es mejor usarlo* |
| `nmap -n <IP>`        | Sin DNS: No resuelve nombres. Ahorra tiempo y evita ruido en los logs de DNS.                  |

## 2. Enumeración Profunda (Scanning)

*Se realiza cuando ya tenemos una IP para analizar*

### Enumeración de Puertos de una **IP** 

| COMANDO                                                       | ESCENARIO                                                                         |
| ------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| `sudo nmap -sS -p- --open -n -Pn -T 4 <IP> -vv -oG Port_scan` | Queremos descubrir todos los puertos abiertos de una IP                           |
| `nmap -sT -p- --open -n -Pn -T 4 <IP> -vv -oG Port_scan`      | Ocupamos `-sT`cuando no tenemos privilegios **root**. *MUY UTIL CON PROXY CHAINS* |

**En caso de no encontrar lanzamos un escaneo UDP o revisamos la sección de Evasión de Firewall**

| COMANDO                         | ESCNEARIO                                                                                                                    |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `nmap -sU --top-ports 100 <IP>` | Escanea solo los 100 puertos UDP más comunes (recomendado porque el UDP completo es lentísimo).                              |
| `nmap -sU -p 161,53,69 <IP>`    | Específico: Vas directo a por SNMP, DNS y TFTP.                                                                              |
| `nmap -sU -sV -p <ports> <IP>`  | Si sale open\|filtered, intenta añadir -sV para que Nmap intente forzar una respuesta y confirmar si está realmente abierto. |
### Enumeración de servicios 

**Una vez que ya encontramos los puertos expuestos de una IP** 

| COMANDO                                                           | ESCENARIO                                                                                               |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `nmap -sCV -p <puertos_encontrados> -Pn -n <IP> -oN Service_scan` | Una vez que encontramos los puertos expuestos queremos descubrir que servicios corren y sus versiones.  |

### Scripts de nmap 

**Nmap tiene scripts especificos utiles para la enumeración de sistemas**. Una vez que ya descubrimos ciertos servicios podemos utilizar de estos scripts antes de utilizar otras herramientas más intrusivas. 

**Los scripts se agrupan según su propósito, las siguientes categorias son las más usadas:**

| CATEGORÍA       | DESCRIPCIÓN                                                       | RIESGO |
| --------------- | ----------------------------------------------------------------- | ------ |
| auth            | Intenta saltarse o probar credenciales en los servicios.          | BAJO   |
| default (`-sC`) | Scripts básicos, rápidos y seguros que Nmap considera esenciales. | BAJO   |
| discovery       | Busca más información del host.                                   | BAJO   |
| vuln            | Busca vulnerabilidads (como EternalBlue). *Muy Útil*              | MEDIO  |
| safe            | Scripts que no deberian ser intrusivos ni tirar el servidor       | BAJO   |
| intrusive       | Scripts que podrían saturar el sistema o ser detectados.          | ALTO   |
| brute           | Realiza ataques de fuerza bruta                                   | ALTO   |
**Los scripts se invocan de 2 formas**

Por nombre individual: 

```bash
nmap --script ftp-anon -p 21 <IP>
```
 
 (Comprueba si el FTP permite entrar sin contraseña).

Por categoría: 
```bash
nmap --script vuln -p 445 <IP> 
```

(Ejecuta todos los scripts de la categoría "vuln" para el puerto 445)

#### Scripts más usados:

#####  SMB (Puertos 139, 445)

SMB suele ser la mina de oro. Quieres saber si hay carpetas compartidas sin contraseña o vulnerabilidades críticas.

```bash
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-vuln-ms17-010 <IP>
```

**Objetivo:** Encontrar carpetas abiertas y verificar si es vulnerable a EternalBlue.

##### HTTP/HTTPS (Puertos 80, 443, 8080)

Si ves un servidor web, quieres saber qué carpetas hay ocultas antes de usar herramientas como GoBuster.

```bash
nmap -p 80 --script http-enum,http-title,http-methods <IP>
```

**Objetivo:** Ver el título de la web, métodos permitidos (PUT, DELETE) y directorios comunes.

##### SSH (Puerto 22)

No suele tener muchas vulnerabilidades de software, pero sí de configuración.

```bash
nmap -p 22 --script ssh-auth-methods,ssh-hostkey <IP>
```

**Objetivo:** Ver qué métodos de autenticación acepta (password, publickey).

#### Busqueda de scripts:

Imagina que estás frente a un servidor Redis y quieres ver qué scripts hay disponibles para él:

```bash
ls /usr/share/nmap/scripts/ | grep "redis"
```

Si queremos buscar un script relacionado a una vuln especifica:

```bash
ls /usr/share/nmap/scripts/ | grep "smb-vuln"
```

Si queremos buscar por categoría específica:

```bash
grep "vuln" /usr/share/nmap/scripts/script.db
```

*'Vuln'  lo cambiamos por la categoría deseada*

### Detección de Firewall

Para la detección de Firewalls, nos vamos a basar en **los estados de respuesta de nmap**.

#### Los 3 Estados clave:

- **Open (Abierto):** La aplicación aceptó la conexión. No hay firewall, o el firewall permite el paso.

- **Closed (Cerrado):** El host respondió con un paquete (RST), diciendo "aquí no hay nada", pero el paquete llegó. Significa que no hay un firewall bloqueando el paso, simplemente el servicio no está activo.

- **Filtered (Filtrado):** Aquí es donde está el firewall. Nmap no recibe respuesta o recibe un error de tipo "ICMP unreachable". El firewall tiró el paquete a la basura (DROP) o lo bloqueó activamente (REJECT).

A pesar de que hay más estados estos son los que nos interesan. 

#### Escaneo ACK

```bash
nmap -sA -p 22,80 -n -Pn <Ip>
```

**El escaneo ACK** es la herramienta principal para la detección de **[[Firewall]]**.   A diferencia de los otros escaneos, este va a intentar determinar si el puerto está filtrado o no filtrado.  nmap envía paquetes **ACK** en lugar de **SYN**.  Al nunca haber entablado una conexión con la víctima, el sístema rechaza automaticamente el paquete **ACK** 

- **Lógica:** Si el host respondé con un RST el paquete ACK llegó al sistema operativo, por tanto, nos devolvería un **Unfiltered**. Eso, quiere decir que no hay un Firewall de por medio. Por el contrario, si el escaneo nos devuelve un **Filtered** quiere decir que el paquete no llegó al sistema, por tanto, hay un **Firewall de por medio**. 

### Evasión de Firewalls y WAF's

**Los firewalls suelen bloquear paquetes basados en reglas de IP, puerto o tipo de protocolo.** 
Las técnicas más utilizadas para evadir estas medidas son:

#### Fragmentación de Paquetes 

Divide las cabeceras TCP en trozos más pequeños. Esto complica que algunos firewalls antiguos o mal configurados reconstruyan el paquete para inspeccionarlo.

```bash
nmap -f <IP>
```

**La opción -f utiliza trozos de 8 bytes y son acumulables**. Si utilizamos -ff utilizariamos paquetes de 16 bytes. 

##### Fragmentación de paquetes avanzada

```bash
nmap --mtu 24 <IP>
```

**Funcionamiento:** Te permite especificar exactamente cuántos bytes quieres que tenga cada fragmento.

**Restricción:** El número que elijas debe ser múltiplo de 8 (ej. 8, 16, 24, 32...). Esto es por cómo funciona el protocolo IP (el desplazamiento de fragmento se mide en unidades de 8 bytes).

**Uso:** Se utiliza cuando sabes (o sospechas) que el firewall tiene una regla específica que bloquea paquetes de 8 bytes pero quizás deja pasar los de 24, o cuando necesitas ajustar el escaneo a las condiciones específicas de esa red.

#### Uso de Decoys o "Señuelos" (-D)

Hacemos que parezca que el escaneo proviene de múltiples direcciones IP a la vez. **Nuestra IP se mezcla entre las falsas**, haciendo que el administrador del sistema no sepa quién es el verdadero intruso.

**Decoys RANDOM**

```bash
 nmap -D RND:10 <IP> 
    ```

(Genera 10 IPs aleatorias como señuelo).

**Decoys con IP's especificas** por ejemplo en redes internas. 

```bash
nmap -D 192.168.1.5,192.168.1.10,ME <IP> 
```

(Usa IPs específicas y ME es tu IP).

#### Suplantación de Puerto de Origen (--source-port)

Muchos firewalls permiten todo el tráfico que venga de puertos "confiables" como el 53 (DNS) o el 80 (HTTP) para no romper la navegación de los usuarios. Puedes obligar a Nmap a enviar paquetes desde esos puertos.

```bash
nmap --source-port 53 <IP>
```

#### Modificar el "Data Length" (--data-length)

Los paquetes de Nmap tienen un tamaño predeterminado muy reconocible por los IDS (Sistemas de Detección de Intrusos). Con esto, añades bytes aleatorios para cambiar la "firma" del paquete.

```bash
nmap --data-length 25 <IP>
```



### Optimización de escaneos

#### Plantillas de temporizado

Nmap facilita la vida con 6 plantillas de velocidad preconfiguradas. Es como el modo de conducción de un coche:

- T**0 (Paranoid) y -T1 (Sneaky)**: Extremadamente lentos. Se usan para evadir IDS muy sensibles. Un escaneo puede tardar días. (Casi no se usan en el eJPT).
- **T2 (Polite):** Reduce la velocidad para consumir menos ancho de banda y no tirar el servidor.
- **T3 (Normal):** Es el modo por defecto. Equilibrio total.
- **T4 (Aggressive)**: El estándar de oro para exámenes. Acelera el proceso asumiendo que estás en una red razonablemente rápida y estable.
- **T5 (Insane):** Velocidad máxima. Solo útil en redes locales (LAN) muy rápidas. Corres el riesgo de perder paquetes y obtener resultados falsos porque el servidor no llega a responder a tiempo.

```bash
nmap -sS -p- -T4 <IP>
```

#### --min-rate

Este comando es un "arma de doble filo".

**¿Qué hace?:** Le dice a Nmap: "No envíes menos de X paquetes por segundo".

**¿Por qué es importante?:** A veces Nmap se vuelve "tímido" si detecta un poco de congestión y baja la velocidad drásticamente. Con --min-rate 1000, le obligas a mantener un ritmo constante de 1000 paquetes por segundo.

**Riesgo:** Si pones un número muy alto en una red inestable, Nmap enviará paquetes tan rápido que el host no podrá responderlos todos, y verás puertos como closed o filtered cuando en realidad están open.

```bash
nmap -sS --min-rate 1000 -Pn -n <IP>
```

#### --host-timeout

Nuestro seguro de vida contra el tiempo perdido.

**¿Qué hace?:** Si un host (una IP) no termina de ser escaneado en el tiempo que definas, Nmap lo abandona y sigue con el siguiente.

**Importancia en optimización:** En el eJPT te darán segmentos de red (ej. /24). Si de 254 IPs, hay 5 que tienen un firewall "agresivo" que ralentiza el escaneo, Nmap podría tardar 30 minutos solo en esas 5.

Uso ideal: --host-timeout 15m asegura que ninguna IP te robe más de 15 minutos de tu examen.

### Importar resultados a metasploit

Metasploit tiene una base de datos interna (PostgreSQL).
Al importar los escaneos podemos:

- **Organización:** Puedes ver todos los hosts con el comando:
```msfconsole
msf5> hosts
```


- **Eficiencia:** Puedes ver todos los puertos abiertos con el comando:
```msfconsole
msf5> services
```

- **Automatización:** Puedes usar comandos como hosts -R para añadir automáticamente todas las IPs encontradas a un exploit (en el campo RHOSTS).

#### Cómo importar resultados? 

**Paso 1.**
Primeramente, tenemos que exportar los resultados de nuestro escaneo nmap en formato:
- **-oX** (xml)
- **-oA** (All formtas)

**Paso 2.**
Antes de abrir Metasploit, asegúrate de que la base de datos esté encendida. En Kali Linux, haz esto en tu terminal:

Iniciar el servicio: 

```bash
sudo systemctl start postgresql
```
*En algunas ocasiones si nos da error debemos usar 'service' en lugar de 'systemctl'*

Inicializar la DB (solo si es la primera vez): 

```bash
sudo msfdb init
```

Abrir Metasploit.

**Paso 3.**
Verificamos la conexión a la base de datos:

```msfconsole
msf5> db_status
```

**Importamos nuestro escaneo**

```msfconsole
msf5> db_import /ruta/de/tu/archivo/mi_escaneo.xml
```
