
--- 
- Tags: #HTB #LATEX #Medium #IMAP 
- --- 
#### QUÉ SE EXPLOTA?

Password Guessing
Abusing e-mail service (claws-mail)
Crypto Challenge (Decrypt Secret Message - AES Encrypted)
LaTeX Injection (RCE)
Bypassing rbash (Restricted Bash)
Extracting Credentials from Firefox Profile

#### WRITE UP 

Como siempre primero vamos a empezar haciendo un escaneo **ICMP** para determinar si la máquina está áctiva o si está inactiva, **además podemos aprovechar para determinar el SO a través del TTL**

![[Pasted image 20251218090139.png]]

Ya que dterminamos que la máquina está encendida, proseguimos a utilizar nmap. 

**El objetivo del primer escaneo es determinar TODOS los puertos abiertos** además para este escaneo utilizaremos un **syn scan** 

```bash
sudo nmap -sS --open -p- --min-rate 5500 -n -Pn 10.10.10.120 -vvv -oG GenPort_scan
```

![[Pasted image 20251218090515.png]]

**encontramos los puertos 80,110,143,993,995,10000** 

Una vez que ya tenemos los puertos abiertos **proseguimos a la segunda etapa de escaneo** en esta etapa **nuestro objetivo es detallar más los puertos previamente encontrados. Ocuparemos los scripts básicos de reconocimiento, junto con el de versión** 

```bash
nmap -sCV -p80,110,143,993,995,10000 10.10.10.120 -oN DetPort_scan
```

Mientras finaliza el escaneo, empezamos a analizar la página web corriendo con **whatweb**. Ya que determinamos que el puerto **80** está abierto. 

![[Pasted image 20251218090917.png]]

Termina el escaneo detallado y vemos:

![[Pasted image 20251218091123.png]]

**Proseguimos a inspeccionar la web**. AL entrar a la página nos topamos con un mensaje que dice que no se puede entrar a través de la URL. **Eso nos lleva a agregar el dominio al [[etc-hosts]]**

![[Pasted image 20251218091235.png]]

![[Pasted image 20251218091411.png]]

**Inspeccionamos la página web, pero, realmente no encontramos nada de utilidad.** Proseguimos a inspeccionar los demás puertos **993,995,10000** ya que estaban relacionados con HTTPS. 

**En los puertos 993 y 995** no encontramos nada.

![[Pasted image 20251218093305.png]]

Pero, en el puerto **10000** encontramos un panel de Login. **El panel de Login está relacionado a un servidor [[Webmin]]

![[Pasted image 20251218093426.png]]

Intentamos con algunas cerdenciales por defecto, pero, no logramos nada. 

**Proseguimos a tratar de descubrir posibles directorios con [[wfuzz]]**. 

```bash

wfuzz --hc=404 -w /usr/share/wordlists/dibuster/directory-list-2.3-medium.txt http://chaos.htb/FUZZ
```


![[Pasted image 20251219091814.png]]

No encontramos mucho en cuestiones de posibles directorios. Por tanto proseguimos a buscar sudominios. Igual ocupamos **[[wfuzz]]**

```
wfuzz --hc=404 -w /usr/share/wordlists/seclists/DNS/subdomains-top... -H 'Host: FUZZ.chaos.htb' http://chaos.htb

```

**Nos muestra muchos carácteres similares** por tanto los tenemos que ocultar de igual manera.

![[Pasted image 20251219092337.png]]

 ```bash
 
 --hh=73
 ```

![[Pasted image 20251219092439.png]]

**Se encontraron 2 subdominios** Webmail y Wordpress por lo que los agregamos al **etc-hosts** 

**Además tenemos que recordar también hacer un escaneo de la IP** hay que recordar que **La IP y el dominio no son lo mismo**. Podemos encontrar rutas en la IP que talvez en el dominio no. 

Encontramos, 

![[Pasted image 20251219110116.png]]

2 Posibles rutas con la IP. 

**Investigamos todo lo obtenido anteriormente** 

La ruta **10.10.10.120/wp** es equivalente a la ruta **wordpress.chaos.htb**. Ahí mismo, encontramos un Post que para visuarilzarlo pide una contraseña. 
![[Pasted image 20251219110529.png]]

**Vemos si podemos obtener algún usuario**

![[Pasted image 20251219110600.png]]

**Hay 1 usuario, llamado 'human'** 

Intentamos con la contraseña:

```bash
human
```

Se reveló el secreto. 

**NOTA** : Siempre hay que intentar con contraseñas 'burdas'. 

![[Pasted image 20251219111118.png]]

Obtuvimos un usuario y una constraeña que podemos usar para **webmail**. 

Entramos a **webmail.chaos.htb** e ingresamos. 

**Además ya que tenemos algunas credenciales podríamos intentar reutilizarlas. Recordemos que en el puerto 143, estaba [[IMAP]] corriendo.**

![[Pasted image 20251219112345.png]]

**Para poder conectarnos a [[IMAP]] utilizaremos: [[Claws-mail]]**.

Ya que configuramos **[[Claws-mail]]**, proseguimos a poner el servidor de **chaos.htb** junto con las credenciales previamente obtenidas. 

![[Pasted image 20251219130436.png]]

Dentro del buzón de AYUS, encontramos en **drafts** un archvivo. 

Según el mensaje encontrado:
![[Pasted image 20251219131822.png]]

Lo que encontramos en **[[Claws-mail]]**. Es básicamente lo mismo que encontramos en **webmail**


![[Pasted image 20251219132011.png]]

**De acuerdo al mensaje, el .txt está cifrado y además tiene una contraseña, que es la misma que el nombre**: sahay. 

En el mismo correo, se nos proporciona el codigo de encriptación. Por lo que podemos pedirle a chatgpt que nos cree un codigo, pero, para desencriptar. 

Una vez obtenido, ejecutamos el script hecho con python y obtuvimos:

![[Pasted image 20251219133000.png]]

**Es un codigo en base64** por lo que lo decodificamos.

![[Pasted image 20251219133106.png]]

