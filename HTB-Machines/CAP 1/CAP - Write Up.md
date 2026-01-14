
--- 
- Tags: #Easy #linux #HTB #capabilities #wireshark
- -- 

#### QUÉ SE EXPLOTA?



#### Write UP

##### PRIMERA FASE: Reconocimiento

**Como ya es costumbre vamoa empezar la primer fase de reconocimiento con un escaneo de todos los puertos**. Una vez habiendo hecho esto vamos a proseguir a realizar un **segundo escaneo más detallado** esto sobre los puertos previamente encontrados.

**El primer escaneo lanzamos un SYN SCAN** para revelar todos los puertos, posteriormente lanzamos un escaneo con los **Scripts por defecto de nmap a los puertos previamente encontados**. 

![[Pasted image 20251109090114.png]]

**Prosiguiendo con la etapa de reconocimiento** tiramos un **whatweb** para conocer un poco más a detalle lo que había corriendo en la página pero, no encontramos mucho. 

![[Pasted image 20251109090741.png]]

**Inspeccionando la página vimos que había una sección en la qué podíamos listar otros datos a través de la url** 

![[Pasted image 20251109092705.png]]

![[Pasted image 20251109093919.png]]

##### SEGUNDA FASE: Explotación

**Encontramos algo desde la ruta 0** además en cada ruta había un botón que descargaba archivos **.cap**
**Proseguimos a analizarlo con [[Wireshark]]** 

![[Pasted image 20251109094527.png]]

**Encontramos un usuario y una contraseña por lo que proseguimos a usarlos en [[SSH]]**.

**Funcionaron las claves.** 

![[Pasted image 20251109095822.png]]


##### TERCERA FASE: Postexplotación - PRIVESC

**Buscamos con [[Comandos utiles para escalar]]** y encontramos qué había **[[Capabilities]]** disponibles para escalar privilegios.

![[Pasted image 20251109100511.png]]

**Python podía cambiar de UID**. Primero importamos la [[libreria OS]] dentro del **interactivo de python3**.  
![[Pasted image 20251109100940.png]]
**Ejecutamos la bash y listo**

![[Pasted image 20251109101013.png]]