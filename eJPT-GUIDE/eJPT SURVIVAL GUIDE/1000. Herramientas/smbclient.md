
--- 
- Tags: #Herramienta #SMB #Enumeration 
- --- 
## Qué es?

**smbclient** es parte de la suite oficial de Samba. 
Es, esencialmente, un cliente similar a un cliente FTP que te permite interactuar directamente con el sistema de archivos remoto. 

**Es la herramienta que usaremos cuando ya sepamos con que *share* vamos a interactuar**

## Cómo se usa?

### Listar recursos:

Antes de entrar, listamos que recursos hay disponibles.

```bash
smbclient -L //<IP>/ -N
```

`-L` : Listar.

`-N` : No pass (intenta sesión nula/invitado).

### Conectarse a un share:

Una vez que ya tenemos algún **share** interesante que queramos explorar. 

```bash
smbclient //<IP>/share -N
```

### Conectarse con creds:

Si tenemos credenciales.

```bash
smbclient //<IP_OBJETIVO>/Backup -U "usuario%password"
```


### Descarga masiva

A veces entras en un share con cientos de archivos y no quieres ir uno por uno. Para descargar todo lo que hay en un share de una sola vez sin que te pregunte archivo por archivo, usa este flujo dentro de smbclient:

    smb: \> recurse ON

    smb: \> prompt OFF

    smb: \> mget *

Esto replicará la estructura de carpetas del servidor en tu máquina local.
### Comandos Internos

Una vez dentro del prompt smb: \>, puedes usar los siguientes comandos:

**Navegación básica:**

`ls` : Listar archivos y carpetas.

`cd <carpeta>` : Entrar en un directorio.

`pwd` : Mostrar la ruta actual en el servidor.

**Gestión de archivos:**

`get <archivo>` : Descarga un archivo a tu máquina local.

`put <archivo>` : Sube un archivo de tu máquina al servidor.

`del <archivo>` : Borra un archivo (si tienes permisos de escritura).

`mkdir <nombre>` : Crea una carpeta nueva.
