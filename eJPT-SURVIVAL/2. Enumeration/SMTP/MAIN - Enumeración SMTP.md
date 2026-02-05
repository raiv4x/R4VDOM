
---
- Tags: #Enumeration #Protocol #SMTP #TCP 
- --

## Qué es?

El **Protocolo Simple de Transferencia de Correo** es el estándar de internet para el intercambio de correos electrónicos. Funciona en la capa de aplicación y, por defecto, utiliza el **puerto 25**.

Su funcionamiento se basa en un modelo cliente-servidor donde se intercambian comandos de texto plano. Es precisamente esa naturaleza de "texto plano" la que nos permite interactuar con él manualmente.

## 1. Telnet y [[Netcat]]

Simplemente es un protocolo de red que nos va a permitir comunicarnos con otra computadora de manera remota, esto lo hace utilizando el protocolo **TCP**. Una vez conectados al servidor se verá con otra entrada. 

```bash
telnet 192.168.1.50 25
```

```bash
nc -nv 192.168.1.50 25
```


![[Pasted image 20260109172614.png]]

**En este caso para enumerar SMTP** 

Para enumerar usuarios, aprovechamos tres comandos específicos que algunos servidores mal configurados mantienen activos:

1. **VRFY (Verify):** Confirma si un nombre de usuario existe.

2. **EXPN (Expand):** Revela la lista de correo real detrás de un alias o grupo.

3. **RCPT TO:** Se usa durante el envío de un correo para identificar destinatarios válidos.

**En algunas ocasiones necesitaremos confirmar que comandos están activos con:**

```bash
HELO attacker.xyz
EHLO attacker.xyz
```


Si el servidor responde `250`, el usuario **existe**. Si responde `550`, **no existe**.

![[Pasted image 20260109173227.png]]

## 2. smtp-user-enum

**smtp-user-enum**, es una herramienta que nos va a servir para enumerar usuarios validos de manera automática empleando un ataque de fuerza bruta

 ```bash
 smtp-user-enum -M [MODO] -U [DICCIONARIO] -t [IP_OBJETIVO]
 ```

**Parametros:**

- **`-M` (Method):** Define qué comando usar (`VRFY`, `EXPN` o `RCPT`).

- **`-U` (User list):** La ruta a un archivo de texto con nombres de usuario (ej. `/usr/share/wordlists/metasploit/unix_users.txt`).

- **`-t` (Target):** La dirección IP del servidor víctima.

- **`-D` (Domain):** A veces necesario si el servidor maneja varios dominios (ej. `objetivo.com`).