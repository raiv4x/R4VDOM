
---
- Tags: #Network #Protocol #Enumeration #Exploitation 
--- 
# Qué es? 

**SNMP** (Simple Network Management Protocol). **Nos sirve para:**

- Monitorear dispositivos de red
- Obtener info de:
    - Interfaces
    - Procesos
    - Usuarios
    - Sistema operativo
    - Routing
    - Servicios

**El puerto 21** es por el que corre **SNMP**

## Community Strings:

### Qué son?

En **SNMP** las community strings son **credenciales de acceso**, equivalen a una contraseña para poder consultar o modificar información via **SNMP**. 

# Enumeración 

Si vemos que el puerto **161** está abierto... 

```bash
nmap -sU -p161 --script snmp-brute target
```

nmap nos proporcionará mayor información sobre el objetivo. 

**Una vez** que ya tenemos mayor información podemos intentar conectarnos:

```bash
snmpwalk -v2c -c public <target>
```

_En caso de que nos arroje mucho ruido podemos meter el contenido a un archivo para despues catearlo_

**Descubrir Usuarios del sistema**

```bash
snmpwalk -v2c -c public target 1.3.6.1.4.1.77.1.2.25
```

**Descubrir interfaces de red**

```bash
snmpwalk -v2c -c public target 1.3.6.1.2.1.2
```

_Si descubrimos más de 1 red, posiblemente podríamos hacer **[[MAIN - Pivoting]]**_

