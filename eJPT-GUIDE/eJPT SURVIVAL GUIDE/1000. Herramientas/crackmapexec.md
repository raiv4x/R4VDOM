
---
- Tags:  #Enumeration #Attacks #Herramienta #SMB #WinRM
- --- 

# Qué es?

**CrackMapExec** es una **navaja suiza principalmente post-explotación** para **redes Windows**.

 No es para “explotar 0days”  
 Es para **validar acceso**, **automatizar ataques** y **movimiento lateral**

Generalmente se usa **después** de obtener:

- Credenciales
- Hashes NTLM
- Acceso parcial a la red

# SMB

## 1. Enumeración
### Sin credenciales

```bash
crackmapexec smb 10.10.10.0/24
```

Obtenemos:
    **Sistema operativo**
    **Dominio**
    **¿SMB signing?**
    **¿SMBv1?**

### Con credenciales

**Enumeración de usuarios**

```bash
crackmapexec smb IP -u user -p pass --users
```

**Enumeración de shares**

```bash
crackmapexec smb IP -u user -p pass --shares
```



## 2. Validación de Contraseñas/Hashes

Cuando obtengamos alguna contraseña, utilizaremos **[[crackmapexec]]** para validar que el usuario y la contraseña sean validos y además **el usuario tenga privilegios**. 

**Con contraseña**

 ```bash
 crackmapexec smb IP -u user -p password
 ```

**Con Hash** [[Pass-The-Hash]]

```bash
`crackmapexec smb IP -u user -H NTHASH`
```

**Pwn3d! = admin local**

# WinRM

## 1. Enumeración

```bash
cme winrm <IP>
```

## 2. Validación Usuario con contraseña/Hash

```bash
cme winrm <IP> -u usuario -p password
```

```bash
cme winrm <IP> -u usuario -H <NTLM>
```

**Además también hará bruteforce si le pasamos un diccionario** 

## 3. Ejecución de comandos

```bash
cme winrm <IP> -u usuario -p password -x "whoami"
```

