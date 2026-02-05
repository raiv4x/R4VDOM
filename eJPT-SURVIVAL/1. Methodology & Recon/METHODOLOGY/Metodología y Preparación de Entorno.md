
---
- Tags: #Methodolgy
- --- 

## 1. Preparación del Entorno

**Antes de hacer cualquier escaneo** lo primero que debemos preparar es nuestro arbol de trabajo. No queremos mezclar archivos de diferentes máquinas
 
**Tenemos que crear una carpeta por cada IP o máquina objetivo**. 

```bash
mkdir -p eJPT/target_1/{scans,exploits,content,others}
```

- scans = Outputs de escaneos (nmap)
- exploits = Para scripts y exploits descargables
- content = Para archivos importantes de la víctima
- evidence = Para capturas de pantalla de flagas y otros archivos importantes.

## 2. RAVX Pentest Methodology

**[[ RAVX CHECKLIST]]** 