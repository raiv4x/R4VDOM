
---
- Tags: #XXE #OWASPTOP10 #HTB #sudo #phpwrapper #Easy 
--- 
#### QUÉ SE EXPLOTA?

 [[XXE]] (XML External Entity Injection) Exploitation XXE PHP File Read - Base64 Wrapper Abusing Sudoers Privilege 

#### Write UP

##### PRIMERA FASE: Escaneos y Reconocimiento

Empezamos haciendo un escaneo como siempre, recordando que l**o primero es descubrir qué puertos** están disponibles en la maquina.  Empleamos un **escaneo STEALTH** para hacer un escaneo más rápido y sigiloso, además de eso, **nos interesa saber únicamente los puertos abiertos**. 

![](../imgs/Pasted image 20251018115428.png)


Posteriormente **hacemos un escaneo más detallado** con los scripts de nmap por defecto, además también empleamos el script de **version**.  

![](../imgs/Pasted image 20251018115412.png)

Encontramos que el puerto **22 y 80** estaban abiertos. Por lo que ahora, proseguimos a ver que codename tienen. **Esto es importante ya que algunos servicios al buscar por [LaunchPad]() los codenames resultan ser distintos, por lo que nose lleva a pensar que estamos frente a un contenedor y no la maquina en si, por la diferencia de codenames**

![](../imgs/Pasted image 20251018115519.png)

![](../imgs/Pasted image 20251018115601.png)

Una vez sabiendo todo esto, nos dirigimos a inspeccionar la pagina, inspeccionando vemos que en la pagina realmente no hay mucho, 

Lo primero que vemos es que hay un recurso que se llama **portal.php** en ese mismo sitio encontramos un botón que nos llevaba a ...**log_submit.php**

![](../imgs/Pasted image 20251018120227.png)


sin embargo, encontramos un recurso **BOunty Report system** que tenía campos donde nosotros **introducimos texto** y se ve representado en la pantalla, por lo que pensamos en [[SSTI]]. Sin embargo, no funcionó. 

##### SEGUNDA FASE: Explotación

Queremos saber como se ve la petición de los campos de entrada, por lo que ocupamos **[[Burpsuite]]**. Lo ponemos como proxy y conseguimos lo siguiente:

![](../imgs/Pasted image 20251018120344.png)
*Ya están decodeados los datos que aparecen en el campo "data"*

Vemos que la petición es una petición **POST** y como campo lleva un parametro llamado **data** y un codigo que parece ser base 64, por lo que proseguimos a **decodearlo** en el mismo [[Burpsuite]]. 

Al decodificarlo, encontramos que nos encontramos ante un **procesado tipo [[XML]]**. Esto nos lleva a intentar un **[[XXE]]**. Hay que URL encodear los campos. 

![](../imgs/Pasted image 20251018120458.png)

![](../imgs/Pasted image 20251018122835.png)

La inyección es positiva por lo que podemos **listar contenido delicado, por ejemplo usuarios disponibles** 

![](../imgs/Pasted image 20251018122947.png)

Encontramos en el [[etc-passwd]] un usuario llamado **development** por lo que proseguimos a guardarlo. 

![](../imgs/Pasted image 20251018123028.png)

**Teniendo en cuenta que hay un [[XXE]] podemos proseguir a ver si hay otros documentos .php en el sistema para ver su contenido usando [[wrappers]]**

En este caso utilizamos el siguiente:

```php
php://filter/convert.base64-encode/resource=archivo.txt
```

En lugar de "archivo.txt" puntamos hacia el mismo **.php** del bounty tracker y funciona. 

![](../imgs/Pasted image 20251018124015.png)

Lo Decodeamos y nos devuelve:

![](../imgs/Pasted image 20251018124036.png)

Empezamos aplicando [[Fuzzing]] para descubrir más archivos **.php** y poder ver si podemos obtener mayor información. 

```bash

wfuzz -c --hc=404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.100/FUZZ.php
```

![](../imgs/Pasted image 20251018124153.png)

![](../imgs/Pasted image 20251018124538.png)

Descubrimos otros archivos **.php**, por lo que proseguimos a decodearlos 
![](../imgs/Pasted image 20251018125626.png)

![](../imgs/Pasted image 20251018125637.png)



mediante la **[[XXE]]** y en uno de ellos encontramos unas **credenciales** por lo que proseguimos anotarlos en nuestro archivo de credenciales. 

**Ahora ya contamos con 2 posibles usuarios y 1 contraseña** por lo que intentamos conectarnos a **SSH** y lo logramos. 

##### TERCERA FASE: Post Explotación

Empezamos por ver lo que había en el directorio **home** sin embargo, solo estaba la flag y otro doc que decía que se nos habían configurado algunos privilegios especiales. 

Checamos los grupos a los que pertenecemos con `id` y no encontramos mucho, sin embargo, al ejecutar `sudo -l`encontramos que podemos ejecutar un script de bash.

![](../imgs/Pasted image 20251018130643.png)

Proseguimos a inspeccionar el script al cual nosotros tenemos capacidad de ejecución como **root** sin embargo, **no lo podemos modificar porque el propietario del archivo es root y no somos root** 

![](../imgs/Pasted image 20251018130736.png)

###### PRIVESC:

Encontramos un script de python que es para validar tickets. **Tenemos que crear un ticket, siguiendo las condiciones del script para poder llegar a la parte del [[eval()]]**. 
![](../imgs/Pasted image 20251018131634.png)
![](../imgs/Pasted image 20251018131608.png)
![](../imgs/Pasted image 20251018133535.png)
Habiendo conseguido llegar al fujo del programa que eval ejecuta podemos simplemente **cambiarle los permisos a /bin/bash** y ganamos **SUID**

![](../imgs/Pasted image 20251018134035.png)
![](../imgs/Pasted image 20251018134059.png)

Y somos Root. 


