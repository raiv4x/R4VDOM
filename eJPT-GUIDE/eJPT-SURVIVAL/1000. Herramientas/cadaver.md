
---
- Tags: #Windows #https #Http #Server #Exploitation #WebDAV
- ---
# Qué es?

**Cadaver** es una herramienta que va a servir como cliente ante un servidor **[[Microsoft IIS]]** con el servicio **[[Explotación WebDAV]]** corriendo. 

**El flujo es:**

```bash
Detecto WebDAV
→ Uso davtest (qué funciona)
→ Uso cadaver (subo shell real)
→ Ejecuto
→ Obtengo acceso
```

# Cómo usar?

```bash
cadaver http://<IP>/webdav/
```

nos dará una shell como:

 ```bash
 dav:/>
 ```

**Una vez dentro podemos ejecutar comandos como:**

```bash
ls
put shell.aspx
get file.txt
delete file.txt
exit
```

