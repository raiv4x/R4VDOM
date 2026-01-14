
---
- Tags: #HTB #Easy #linux #MYSQL
--- 
#### QUÉ SE EXPLOTA?

Virtual Hosting Enumeration
Abusing Support Ticket System
Access to MatterMost
Information Leakage
Database Enumeration - MYSQL
Cracking Hashes
Playing with hashcat rules in order to create passwords
Playing with sucrack to find out a user's password

#### WRITE UP

Al igual que siempre **vamos a comenzar realizando nuestros 2 escaneos de rutina**. 

El primer escaneo **vamos a emplear un SYN scan con la finalidad de descubrir todos los puertos abiertos**. 
![[Pasted image 20251110151127.png]]

Posteriormente lanzaremos un segundo escaneo, **el segundo escaneo tendrá como objetivo obtener detalles sobre los puertos previamente escaneados** 

![[Pasted image 20251110151208.png]]

**Además de eso, seguimos la etapa de reconocimiento** lanzando un **whatweb**, esto para obtener mayor información sobre la página corriendo en el **puerto 80**. 

![[Pasted image 20251110151437.png]]
**No encontramos nada útil**. Por lo que proseguimos a inspeccionar la página web. En la página web, nos salen un boton de soporte

![[Pasted image 20251110170823.png]]
**Investigando túvimos que agregar 2 dominios al [[etc-hosts]]**. Esto de igual manera lo pudimos haber descubierto con [[GoBuster]], pero en este caso estaban en la página. 

Una vez que agregamos los 2 dominios **proseguimos a seguir investigando**.

![[Pasted image 20251111071718.png]]
![[Pasted image 20251111072313.png]]
El primer subdominio era el **helpdesk** el cual nos permitía logearnos, crear un ticket o una cuenta. **al no tener credenciales, decidimos ver si podíamos crear una cuenta**. Al crear cuenta se nos manda correo de confirmación, lo cuál es sinonimo de que por ahí no es. 

**Además estaba esa parte de contacto que nos decía que con un email delivery.htb podriamos acceder al servidor de [[Mattermost]]**

![[Pasted image 20251111072251.png]]
![[Pasted image 20251111072706.png]]
Dentro de **[[Mattermost]]** vimos que teniamos que crear una cuenta o tener una, sin embargo **igual nos iban a mandar un correo de verificación** por lo que no tenía sentido seguir.

**Seguimos inspeccionando, y nos creamos un ticket** este Ticket lo creamos como invitado. 
![[Pasted image 20251111073226.png]]
Al crear un ticket nos da un **número de ticket** además nos dice que **podemos añadir más información al ticket, mandando la información a un correo que se nos creó**. 

Al checar el ticket con el nnúmero que se nos dió y además el correo que introdujimos se nos abre el seguimiento del ticket. 

![[Pasted image 20251111073906.png]]

##### SEGUNDA FASE: Explotación

**En teoría estamos dentro del correo del ticket que se nos creó** por lo que podríamos usar ese **'correo'** para recibir la validación de [[Mattermost]]. **Usamos el correo del ticket y listo**

![[Pasted image 20251111074309.png]]

**Recibimos confirmación**

![[Pasted image 20251111074351.png]]

**Verificamos la cuenta entrando a la *url* que se nos proporcionó** entramos a nuestra cuenta y listo, tenemos acceso a [[Mattermost]].

**Dentro de [[Mattermost]]** encontramos filtradas un usuario y una contraseña. Por lo que decidimos probarla en **[[SSH]]**.

![[Pasted image 20251111075257.png]]

![[Pasted image 20251111075412.png]]

##### TERCERA FASE: Post Explotación - Privesc

**Empezamos buscando con los [[Comandos utiles para escalar]]** y encontramos en los **pŕocesos** algo interesante:

![[Pasted image 20251111130822.png]]

**Hay una base de datos [[MariaDB]] corriendo** que probablemente sea la base de datos de [[Mattermost]]. **Además encontramos** que estaba corriendo [[Mattermost]] y la ruta, por lo que decidimos ver si posiblemente ahí había algún **archivo de configuración ya que en ellos podemos encontrar usuarios y contraseñas** 

Nos dirijimos a la ruta de **mattermost** y... **carpeta config**

![[Pasted image 20251111131049.png]]
Dentro de ella encontramos un archivo **[[json]]** que contenía una clave para la base de datos.

![[Pasted image 20251111131539.png]]

**Nos conectamos a [[MariaDB]] con el usuario encontrado y ahí encontramos una tabla con usuarios y contraseñas hasheadas** 

![[Pasted image 20251111131922.png]]

Primeramente con [[hashid]] vemos que tipo de hash tenemos. 

![[Pasted image 20251111132614.png]]

**Recordemos que en la página donde encontramos las credenciales ssh** había una nota que decía que la contraseña no se encontraba en el rockyou, sin embargo, con **reglas de [[hashcat]]** podíamos dar con la contraseña. 

![[Pasted image 20251111132933.png]]

![[Pasted image 20251111133842.png]]
Creamos el diccionario con **hashcat** y lo utilizamos para crackear la contraseña. 

![[Pasted image 20251111134307.png]]

**Tenemos la contraseña de root**

![[Pasted image 20251111134429.png]]

**listo**

