
--- 
- Tags: #Enumeration #FTP #Metasploit #Protocol 
- --- 

## Qué es FTP? 

**FTP** (File Transfer Protocol), es **un protocolo de red** diseñado específicamente para la transferencia de archivos entre un cliente y un servidor a través de una red TCP/IP (como Internet).

## Conceptos clave

**1.FTP utiliza arquitectura de Doble Canal**

A diferencia de otros protocolos, FTP no usa una sola conexión. Abre dos canales distintos:

- **Canal de Control (Puerto 21):** Se usa para enviar comandos (ej. ls, get, put) y autenticarse. Es donde ocurre la "charla" entre cliente y servidor.

- **Canal de Datos (Puerto 20 en modo activo):** Se usa exclusivamente para mover el contenido de los archivos.

**2.Modos de Conexión: Activo vs. Pasivo**

Esto es vital cuando haces auditorías detrás de firewalls:

**Modo Activo:** El cliente abre un puerto y el servidor inicia la conexión hacia el cliente para transferir datos. (Suele fallar si el cliente tiene firewall).

**Modo Pasivo:** El cliente inicia ambas conexiones. Es el estándar hoy en día porque evita problemas con firewalls locales.

**3.Riesgos**

FTP original no cifra nada.

**Los usuarios y contraseñas viajan en texto plano.**

Si un atacante hace un sniffing (con Wireshark o Ettercap) en la red, puede ver las credenciales sin esfuerzo.

Para solucionar esto, existen FTPS (FTP sobre SSL/TLS) y SFTP (que usa SSH y es un protocolo completamente diferente, aunque sirva para lo mismo).

## 1. Escaneo de Versión y Banner

Utilizaremos los módulos auxiliares de **[[Metasploit]]**

Antes de lanzar exploits, necesitamos saber a qué nos enfrentamos. **El banner de bienvenida de FTP a menudo revela el software y la versión exacta.**

**Módulo:** 

```msfconsole
msf5> search auxiliary/scanner/ftp/ftp_version
```

**Objetivo:** Identificar si el servidor es vsftpd, ProFTPD, Pure-FTPd, etc.

## 2. Comprobación de Acceso Anónimo

Uno de los fallos de configuración más comunes es permitir el acceso con el usuario anonymous sin contraseña (o con un correo falso).

**Módulo:** 
```msfconsole
msf5> auxiliary/scanner/ftp/anonymous
```

**Qué buscar:** Si el acceso es de lectura/escritura (READ/WRITE). Si tienes permisos de escritura, podrías subir un payload o una shell reversa.

## 3. Fuerza Bruta de Credenciales y [[Metasploit]]

Si el acceso anónimo está desactivado, el siguiente paso lógico es intentar descubrir credenciales válidas mediante diccionarios.

**Módulo**:
```msfconsole
msf5> auxiliary/scanner/ftp/ftp_login
```
**Parámetros clave:**

**USER_FILE**: Ruta a tu diccionario de usuarios (ej./usr/share/wordlists/metasploit/namelist.txt).

**PASS_FILE**: Ruta a tu diccionario de contraseñas (ej. Rockyou).

**STOP_ON_SUCCESS**: Establécelo en true para detenerte al hallar una cuenta.}

## 4. Fuerza Bruta con [[hydra]]

Si ya conocemos el **usuario** (por ejemplo, `admin`) y queremos probar una lista de **contraseñas**, el comando se ve así:

```bash
hydra  -t 32 -l admin -P /ruta/a/tu/diccionario.txt ftp://192.168.1.10
```

- **`-t 1-64`**: Se usa para indicar los hilos, **64 el maximo** 
- **`-l` (l minúscula):** Se usa cuando conoces el nombre de **usuario** específico.
- **`-L` (L mayúscula):** Se usa si quieres probar una **lista de usuarios** desde un archivo.
- **`-p` (p minúscula):** Se usa si conoces la **contraseña** y quieres probar varios usuarios.
- **`-P` (P mayúscula):** Indica la ruta al **diccionario de contraseñas** (ej. `/usr/share/wordlists/rockyou.txt`).
- **`-vV`:** (Opcional) Activa el modo _muy detallado_. Verás en pantalla cada intento que hace Hydra (usuario:contraseña).
- **`-f`:** (Opcional) Detiene el ataque en cuanto encuentra la primera combinación válida.
- **`ftp://[IP]`:** Especificas el protocolo y la dirección del objetivo.

