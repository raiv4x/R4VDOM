
---
- Tags: #Herramienta #SMB #Enumeration 
- ---

## Qué es? 

**Es una herramienta básica** dentro del mundo del hacking. Esta herramienta nos va a permitir recolectar datos a través de un servicio **SMB**

**enum4linux** es un wrapper, es decir una herramienta que utiliza otras herramientas (smbclient,rpcclient, nblookup, entre otras)

## Cómo usarlo? 

**Escaneo General**

```bash
enum4linux -a <IP>
```

El flag -a (all) intenta absolutamente todo: usuarios, grupos, shares, políticas y SIDs. *Es el que se ocupa la mayor parte del tiempo*.

**Más escaneos**:

`-U` : Solamente obtiene la lista de usuarios. 

`-S`: Solamente obtiene la lista de shares

`-P` : Solamente obtiene la política de contraseñas.

`-r` : Enumerar usuarios vía **RID cycling**

		Recordemos: RID=Relative Identifier
		¿Qué es?: En Windows, un usuario tiene un SID (Security Identifier). El RID es la parte final. Por ejemplo, el Administrador siempre termina en 500. Enum4linux prueba del 500 al 2000 para ver qué nombres de usuario le devuelve el servidor.


