
--- 
- Tags: #Methodolgy #Enumeration 
- --- 
## En qué consiste?

La etapa de enumeración consiste básicamente en **descubrir la mayor información posible sobre los puertos y servicios previamente encontrados.** Esta información nos va a servir para posteriormente explotar las vulnerabilidades encontradas en esta fase. 

## Flujo de Trabajo


El secreto de la eficiencia no es ir rápido, sino no tener que repetir tareas. Este es el orden lógico:
### Fase A: El Escaneo de "Superficie"

Se empieza con algo rápido para identificar qué está vivo.

**Nmap inicial** : Se lanza un -p- (todos los puertos) para asegurar que no todos los puertos son escaneados.

### Fase B: Priorización de Servicios (¿Por dónde empiezo?)

Si encuentras varios servicios, este es el orden de "atención" basado en el valor de la información que proporcionan:

#### Servicios de Red/Infraestructura (DNS, SNMP):

**¿Por qué?** Pueden revelarte otros nombres de máquinas, estructuras de red o versiones de software de todo el parque informático. Un Zone Transfer en DNS es una mina de oro.

#### Servicios de Identidad y Archivos (SMB, LDAP, RPC):

**¿Por qué?** Estos servicios suelen permitir "Null Sessions" (sesiones anónimas). Enumerar usuarios de AD o ver carpetas compartidas sin contraseña te da el 50% del trabajo hecho.

#### Servicios Web (HTTP/HTTPS):

**¿Por qué?** Tienen la superficie de ataque más grande. Aplicaciones personalizadas, plugins desactualizados o archivos de configuración expuestos.

#### Servicios de Administración Remota (SSH, RDP, Telnet):

**¿Por qué?** A menos que veas una versión muy antigua y vulnerable, aquí solo podrás hacer fuerza bruta. Déjalos para el final, cuando ya tengas una lista de usuarios recolectada de los puntos anteriores.
        


