
---
- Tags: #Herramienta #Enumeration #Http #https 
- ---

## Qué es?


**Gobuster** es una herramienta escrita en el lenguaje **Go**. Su función principal es el **fuzzing** (fuerza bruta): toma una lista gigante de palabras (diccionario) y las prueba una por una contra el servidor para ver qué responde.
## Su importancia en la Enumeración Web:

- **Encuentra contenido oculto:** Directorios como `/admin`, `/backup`, o archivos como `config.php` que no están enlazados en el menú principal.

- **Rapidez:** Al estar programado en Go, es extremadamente veloz y puede manejar muchas peticiones simultáneas.

- **Versatilidad:** No solo busca carpetas; también encuentra subdominios y buckets en la nube.

## 1. Busqueda de Directorios. 

Este es el modo que talves más estemos utilizando, además es el más básico. 

```bash
gobuster dir -u http://sitio.com -w /ruta/diccionario.txt
```

`dir` : Indicamos el modo
`-u` : Especificamos la url
`-w` : Indicamos la ruta al diccionario que queremos ocupar. 

## 2. Modo DNS 

Sirve para encontrar subdominios como `dev.sitio.com` o `prueba.sitio.com`.

```bash
gobuster dns -d sitio.com -w /ruta/diccionario.txt
```

`dns` : Indicamos el modo
`-d` : Indicamos el dominio al cual vamos a aplicarle el fuzzing
`-w` : Indicamos la ruta al diccionario que queremos ocupar. 

## 3. Modo VHOSTS

A veces un servidor aloja varios sitios web en la misma IP. Este modo ayuda a identificarlos.

```bash
gobuster vhost -u http://10.10.10.10 -w /usr/share/wordlists/ --append-domain
```

`vhost`: Indicamos el modo
`-u` : Indicamos la IP o dominio
`-w` : Indicamos el diccionario a utilizar
`--append-domain` : Esto hará que gobuster busque `dev.ejemplo.com`


## 4. Temporizado

Para jugar con hilos y que el escaneo vaya más rápido de lo normal usamos..

```bash
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlist/ -t 200
```

|**Configuración**|**Velocidad Aprox.**|**Tiempo Total**|**Uso recomendado**|
|---|---|---|---|
|`-t 1`|Muy lenta|~30 minutos|Evasión máxima de seguridad.|
|`-t 10` (Default)|Moderada|~3 minutos|Escaneo estándar y seguro.|
|`-t 50`|Rápida|~40 segundos|CTFs y máquinas de práctica.|
|`-t 200`|Extrema|~10 segundos|¡Peligro! Solo en redes locales muy robustas.|
## 5. Gestión de Sesiones y Cookies

Al escanear sin cookies seleccionadas podríamos perder directorios exclusivos para usuarios.

```bash
gobuster dir -u http://sitio.com/panel -w dict.txt -c "session=12345abc; user=admin"
```

`-c` : Indicamos la cookie de sesión. 
