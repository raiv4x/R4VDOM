
---
- Tags: #Exploitation #Enumeration #BruteForce #Herramienta 
- ---
## Qué es?

**Hydra** es la herramienta por excelencia utilizada por la mayoría de Hackers y pentesters. Hydra se utiliza para realizar **ataques de fuerza bruta** o **ataques de diccionario** contra servicios que requieren autenticación (usuario y contraseña).

A diferencia de otras herramientas, Hydra destaca por dos razones:

- **Velocidad:** Es "paralela", lo que significa que puede probar muchas contraseñas al mismo tiempo (múltiples conexiones simultáneas) en lugar de una por una.

- **Versatilidad:** Soporta más de **50 protocolos** diferentes. Algunos de los más comunes son:
    
    - **SSH, Telnet, FTP** (Acceso a servidores).
    
    - **HTTP/HTTPS** (Formularios web, inicios de sesión de sitios).
    
    - **MySQL, PostgreSQL, Oracle** (Bases de datos).
    
    - **RDP** (Escritorio remoto de Windows).
    
    - **SMB** (Compartición de archivos).

## 1. Cómo se usa? 

### 1. FTP 

[[MAIN - Enumeración FTP#4. Fuerza Bruta con hydra]]
### 2. MySQL

[[MAIN - Enumeración SQL#3. Fuerza Bruta con hydra]]

### 3. SMB

[[MAIN - Enumeración SMB#4. Fuerza bruta con hydra]]

### 4. SSH

**Hydra** talvez se caracteriza por ser super eficiente en **SSH**

```bash
hydra -L users.txt -P pass.txt -t 4 ssh://10.10.10.50
```

**Definir el puerto (`-s`):** Si el servidor SSH no está en el puerto estándar (22), usa `-s`.


