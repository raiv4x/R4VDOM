
---
- Tags: #Herramienta #Windows #Enumeration #Http #https #WebDAV
- ---

# Qué es? 

Es una herramienta usada para enumerar sistemas que cuentan con el servicio **[[Explotación WebDAV]]** hábilitado. 

**Davtest** nos va a responder las siguientes preguntas de manera rápida:

**¿Puedo escribir?**
**¿Dónde puedo escribir?**
**¿Qué extensiones acepta?**
**¿Se ejecutan?**

# Cómo usar?

**Uso básico**:

```bash
davtest -url http://<IP>/
```

**Uso con Creds**:

```bash
davtest -url http://<IP>/webdav/ -auth user:password

```



