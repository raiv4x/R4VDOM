
--- 
- Tags: #Enumeration #RDP #Metasploit 
- -- 

# Qué es? 

**RDP (Remote Desktop Protocol)** es un **protocolo de Microsoft** que permite **conectarte de forma remota a una computadora Windows** y **controlarla gráficamente**, como si estuvieras sentado frente a ella.

RDP se usa para:

- Administrar **servidores Windows**
- Dar soporte remoto
- Acceso a escritorios corporativos
- Trabajo remoto

**Ejemplo real:**

> Un administrador entra por RDP a un **Windows Server** desde su laptop.

**RDP utiliza el puerto 3389**

# 1. Identificación con Mestasploit

En muchas ocasiones el puerto asignado a **RDP** no será el **3389**. Recordemos que **RDP nos permite cambiar el puerto que trae por defecto** 

```bash
auxiliary/scanner/rdp/rdp_scanner
```
