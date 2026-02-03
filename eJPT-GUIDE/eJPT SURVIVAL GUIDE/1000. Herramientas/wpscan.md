
---
- Tags: #Http #HTTPS #CMS #Herramienta 
---
# Qué es?

Herramienta de **enumeración específica de WordPress**.  
Sirve para descubrir:

- Versión de WordPress
- Plugins
- Temas
- Usuarios
- Vulnerabilidades conocidas
- Configuraciones débiles

# Uso:

**Comando básico**:

```bash
wpscan --url http://target.com
```

Esto ya te devuelve:

- Versión WP
- Tema activo
- Algunos plugins
- Headers interesantes

## Enumeración

### Usuarios

**comando:**

```bash
wpscan --url http://target.com -e u
```
### Plugins

**comando:**

```bash
wpscan --url http://target.com -e p
```

_A veces no detecta nada por tanto ocupamos:_

```bash
wpscan --url http://target.com --plugins-detection aggressive
```

