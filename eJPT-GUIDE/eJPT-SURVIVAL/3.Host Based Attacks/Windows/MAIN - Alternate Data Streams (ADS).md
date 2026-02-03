
--- 
- Tags: #Windows #Archivos
- --- 

# Qué son? 

Las **ADS (Alternate Data Streams)** son una **característica del sistema de archivos NTFS** de Windows.

**Permiten que un archivo tenga más de un contenido interno**, aunque tú solo veas uno.

# 1. Creación

Primeramente **se crea el archivo que ocupara lugar como señuelo.** 

```bash

type null > archivo.txt
```

Posteriormente creamos el **ADS**

```bash

echo "hola mundo" > archivo.txt:secreto.txt

```

# 2. Detección

```cmd
dir /r
```

```cmd

more < archivo.txt:secreto.txt
```

# 3. Ejecutables

```bash
type malware.exe > imagen.jpg:malware.exe
```

**Para ejecutar:**

```bash
start imagen.jpg:malware.exe
```

En algunas ocasiones sino se ejecuta tendremos que crear un link que al ejecutarlo nos accione el **ADS**. 

```cmd
mklink wupdate.exe C:\Windows\temp\windowslog.txt:oculto.exe
```

