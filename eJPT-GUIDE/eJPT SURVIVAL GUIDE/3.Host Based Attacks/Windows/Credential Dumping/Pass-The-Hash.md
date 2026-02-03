
---
- Tags: #Windows #Post-exploitation #Hashes #Credentials 
- --- 

# Qué es?

**Pass-the-Hash** es una técnica que permite **autenticarse en un sistema remoto usando directamente el hash NTLM**, **sin conocer la contraseña en texto plano**.

- Windows acepta hashes NTLM como prueba de identidad.  
- No se crackea nada.  
- Solo se **reutiliza el hash**.

**Se utiliza** cuando se compromete una máquina y se tienen privilegios elevados. **Recordemos, que al comprometer una máquina y escalamos privilegios es a nivel local**. Sin embargo, a veces el admin local no tiene permisos de admin de dominio, por tanto ahí entra en juego **Pass-The-Hash**. 

**Nota:** Los Hashes almacenados únicamente son aquellos de usuarios que han entablado alguna conexión con la máquina comprometida. 

# 1. Hash Dumping

Empezamos utilizando **[[Mimikatz]]**  para dumpear los **Hashes** almacenados. Una vez que ya nos transferimos el ejecutable, y nos dimos permisos:

```cmd
lsadump::sam
```

# 2. Movimiento Lateral 

Una vez que ya obtuvimos el **HASH NTLM** podemos proseguir a usar **[[psexec.py#Pass-The-Hash]]** para reutilizar el **hash** y obtener otra sesión.

```bash
psexec.py CORP/admin@192.168.1.50 -hashes :a4f49c406510bdcab6824ee7c30fd852
```

## Alternativas:

- **[[crackmapexec#SMB#2. Validación de Contraseñas/Hashes]]**
- **[[evil-winrm#Cómo usar]]** (Sí está activo)

