
---
- Tags: #Herramienta #Exploitation #RDP 
- -- 
# Qué es?

**[[xfreerdp]]** es un **cliente RDP de código abierto** (parte del proyecto **FreeRDP**).

Permite:

- Abrir **sesiones gráficas RDP**
- Autenticarse de varias formas
- Usarse en **ataques reales** (PtH, dominios, NLA, etc.)
# Conexión con contraseña

```bash
xfreerdp /v:10.10.10.5 /u:admin /p:Password123
```

# Conexión Passh-The-Hash

```bash
xfreerdp /v:10.10.10.5 /u:administrator /pth:<NTLM_HASH>
```

# Opciones importantes

|Opción|Función|
|---|---|
|`/v:`|IP o hostname|
|`/u:`|Usuario|
|`/p:`|Contraseña|
|`/d:`|Dominio|
|`/pth:`|Pass-the-Hash|
|`/cert:ignore`|Ignorar certificado|
|`/size:`|Resolución|
|`/clipboard`|Copiar/pegar|
