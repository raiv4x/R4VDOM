
---
- Tags: #Enumeration #SMB 
- --- 
## Qué es SMB?

**SMB (siglas de Server Message Block) es un protocolo de red utilizado para compartir archivos, impresoras y puertos serie entre nodos de una red.** Es la base de la comunicación en redes locales (LAN), especialmente en entornos Windows, aunque también está disponible para LINUX como **SAMBA**

**Lo que nos debe importar y tenemos que recordar, a través de SMB podemos acceder a recursos con información sensible**. Además tenemos que recordar:

`Shares = recursos compartidos'

## 1. Enumeración básica

#### 1.  Primera fase:

**Lo primero es confirmar que el servicio está activo**. SMB suele correr en el puerto TCP 445 (Direct Host) o a través de NetBIOS en los puertos 137, 138 y 139.

**Para comenzar** utilizaremos la herramienta y **'navaja suiza'** de **SMB**. 
**[[enum4linux]]**

**enum4linux** se centra en la identidad, es decir, quién está en la red. 

#### 2. Segunda fase

**Una vez teniendo información sobre la red queremos determinar los permisos de los recursos previamente encontrados** 
Para esta etapa utilizaremos: 
**[[smbmap]]**
**[[crackmapexec#Enumeración SMB]]**

**SMBMAP** es la herramienta por antonomacía. **La utilizamos para analizar información sensible de los recursos, como los permisos, además de que interactua con el contenido** 

#### 3. Tercera fase

En esta fase **nos conectaremos directamente al servicio smb**. 
Para esto ocupamos **[[smbclient]]**

La consola interactiva cambia un poco, se tendrá que ver y utilizamos los comandos de **[[smbclient]]**

```smb
smb: \>
```

## 2. Fuzzing de shares

En algunas ocasiones los shares son 'secretos' por lo que **únicamente se pueden acceder con la ruta especifica**. 

Para esto tenemos que hacer un pequeño script que nos permita hacer fuzzing de shares.

**Para realizar un fuzzing de shares ocupamos** nuestra herramienta:

**[[fzshare]]**


