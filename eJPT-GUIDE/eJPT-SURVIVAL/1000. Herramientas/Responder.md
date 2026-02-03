
---
- Tags: #Windows #Attacks #Network #Hashes 
- -- 
# Qué es? 

**Responder** es una herramienta para **envenenar la resolución de nombres en redes Windows** y **capturar credenciales**.

En Windows, cuando una máquina **no puede resolver un nombre por DNS**, intenta otros métodos:

1. **LLMNR** (Link-Local Multicast Name Resolution)
2. **NBT-NS** (NetBIOS Name Service)
3. **MDNS**

**Responder se hace pasar por el servidor que “responde” primero**, diciendo: _“Ese recurso soy yo”_

# Uso

**Primero cambiar** en `Responder.conf`

Porque si no:

- Responder **se queda con la autenticación**
- ntlmrelayx **nunca la ve**
- Solo capturas hashes → **NO relay**

```bash
SMB = Off
HTTP = Off
```

**Lo echamos a andar**

```bash
responder -I eth0
```

