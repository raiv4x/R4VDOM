
---
# Ports

```bash
nmap -p- --open --min-rate 5000 -Pn -n -vv -oG tcp_scan target1.ine.local target2.ine.local
```

![[Pasted image 20260115114921.png]]

**El puerto 80 (Enumeración Web)**

**El puerto 135 (RPC)**

**El puerto 139 y 445 (SMB)**

**El puerto 3389 (RDP)** 

**El puerto 5985 (WinRM)** 

# Service Scan

![[Pasted image 20260115115545.png]]

# SMB ( 139/445)

**Se necesitan credenciales para poder listar**. *De acuerdo al CTF ya tenemos un usuario*

**bob**

Intentamos fuerza bruta con **[[Metasploit]]**

![[Pasted image 20260115120927.png]]
**Tenemos Credenciales**




