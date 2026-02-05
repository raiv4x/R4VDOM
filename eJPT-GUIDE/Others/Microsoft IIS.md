
---
- Tags: #Server #Windows #Http #https #Vulns 
- ---
# Qué es? 

Básicamente **IIS** es el servidor **web** de Microsoft, equivale a **Apache o Nginx** de Linux.  Se usa para albergar sitios **PHP** o **ASP.NET** 

- Corre en **Windows**
- Usa **HTTP / HTTPS**
- Normalmente escucha en:
    - `80`
    - `443`
    - A veces `8080`, `8443`

Soporta archivos:
- **.asp**
- **.aspx**
- **.config**
- **.php**

# Versiones

| IIS     | Windows                  |
| ------- | ------------------------ |
| IIS 6.0 | Windows 2003             |
| IIS 7.x | Windows 2008             |
| IIS 8.x | Windows 2012             |
| IIS 10  | Windows 10 / 2016 / 2019 |
# [[Explotación WebDAV]]

WebDAV es un **módulo/feature opcional de IIS**

Microsoft lo incluye para:
- Administración remota
- Gestión de contenidos

No siempre está habilitado

- El admin debe **activarlo explícitamente**
- En muchos IIS viejos o mal configurados **queda abierto**

**si está mal configurado**:

- Permite **subir archivos ejecutables**
- Permite **web shells**
- Lleva directo a **RCE**