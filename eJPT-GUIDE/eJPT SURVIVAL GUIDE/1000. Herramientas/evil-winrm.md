
---
- Tags: #WinRM #Exploitation #Attacks #Windows 
- ----

# Qué es?

**Evil-WinRM** es una **herramienta de explotación y post-explotación** que te permite obtener una **shell interactiva en Windows** usando el servicio **WinRM (Windows Remote Management)**.

Necesitas **credenciales**, no es una vulnerabilidad por sí misma.
Funciona si el usuario:
- Existe
- Tiene WinRM habilitado
- Pertenece a:

    - `Remote Management Users`
    - o `Administrators`

# Cómo usar:

**Usuario + contraseña**

```bash
evil-winrm -i IP -u usuario -p password
```

**Usuario + hash NTLM [[Pass-The-Hash]]**

```bash
evil-winrm -i IP -u usuario -H <hash>
```

