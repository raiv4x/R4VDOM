
--- 
- Tags: #HTB #Easy #linux #python #sudo #SQLite
- ---

#### QUÉ SE EXPLOTA? 

#### Write - UP

##### PRIMERA FASE: Reconocimiento Áctivo y Pásivo

**Cómo ya es de costumbre** empezamos nuestros 2 escaneos. El primero con un **syn scan** para reconocer todos los puertos abiertos y posteriormente **lanzamos un escaneo detallado de los puertos previamente encontrados** 

![[Pasted image 20251104153501.png]]
**Vemos que a través del puerto 5000 está corriendo un servidor [[Gunicorn]]**. 

Inspeccionando la **web** se descubrió que es **una plataforma que ejecuta codigo python**.

![[Pasted image 20251104160347.png]]

![[Pasted image 20251104160359.png]]


**Tratamos de ver si podiamos importar alguna librería como [[libreria OS]]** para ejecutar comandos a nivel sistema.

![[Pasted image 20251104160608.png]]
![[Pasted image 20251104160550.png]]

##### SEGUNDA FASE: Explotación

**Primero vimos que palabras estaban bloqueadas**:

**Para poder bypassear** esta santitización podemos jugar con los **[[built-in]] de python y utilizando *getattr***. 

```python

import
os 
system
__import__

```

**Para bypassear** aplicamos lo siguiente con **[[getattr()]]** y los **[[built-in]]**.

![[Pasted image 20251107153857.png]]

**Una vez que recibimos la shell empezamos a buscar como escalar privilegios**
![[Pasted image 20251107154200.png]]
##### TERCERA FASE: Post explotación - PRIVESC

**Empezamos a buscar con los [[Comandos utiles para escalar]]** y encontramos que estabamos en el directorio de la página. **checamos si había alguna filtración en la configuración**

Encontramos que había una **base de datos** corriendo por lo que proseguimos a inspeccionarla **ya que había panel para registrarse** por tanto seguramente hay usuarios disponibles.

![[Pasted image 20251107154604.png]]

**Es una [[SQlite]]**

Dentro de la **base de datos** encontramos usuarios y sus hashes. Por lo que proseguimos a verificar que eran en [[crackstation]]

![[Pasted image 20251107154839.png]]

![[Pasted image 20251107154851.png]]


**Proseguimos a conectarnos como el usuario martin por ssh** 
Una vez que nos conectamos **buscamos maneras de escalar privilegios a root** y encontramos que había un **script** que corria como **root** o sea con [[SUDO]]. 

![[Pasted image 20251107155133.png]]

**El script creaba una copia de una ruta que le pasaramos** al analizar el codigo vimos que era vulnerable a **[[Directory Path Traversal]]** por lo que de esa manera podíamos leer cualquier directorio al **modificar los campos requeridos**

![[Pasted image 20251107155320.png]]

**requería un archivo [[json]]**

Había uno ya creado en la carpeta de **backups** por lo que modificamos ese.

![[Pasted image 20251107155456.png]]

**Nos devolvio un archivo .tar** con todo el contendio de /root. **lo descomprimimos con tar -xf** y listo, guardamos el **id_rsa** y nos conectamos como root. 

![[Pasted image 20251107155636.png]]

![[Pasted image 20251107155658.png]]

![[Pasted image 20251107155729.png]]

![[Pasted image 20251107155750.png]]




