
---
- Tags:  #Windows #Post-exploitation #PrivEsc #Attacks 
- --- 
# Qué es?

`akagi64.exe`es una herramienta que pertenece a **[UACme](https://github.com/hfiref0x/UACME)** se usa para:  
👉 **hacer un bypass de UAC**  
👉 **obtener una consola o payload con token de administrador elevado**

**Requisito clave**:

- El usuario **DEBE pertenecer a `Administrators`**
- **NO** debe estar elevado aún

# Cómo usar?

Una vez que ya tenemos sesión en la máquina Victima, tenemos que transferirnos el binario de **akagi64.exe**. Una vez que lo tengamos lo ejecutamos. 

## Antes de usarlo (checklist obligatorio)

Siempre verifica esto primero:

1. **Soy admin**
    
    `whoami /groups`
    → `Administrators` debe aparecer
    
2. **NO estoy elevado**
    
    `whoami /groups`
    
    → `Administrators: Disabled`
    

Si esto no se cumple → **akagi64 no sirve**.

## Sintaxis básica de akagi64

La forma general es:

```bash
akagi64.exe <método> <payload>
```

Donde:

- `<método>` → número del método (ej. 23)
- `<payload>` → lo que quieres ejecutar elevado