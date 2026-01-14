
---
- Tags: #Herramienta #Fuzzing #Enumeration 
- ---
## Qué es?

Al igual que **[[ffuf]]**, **wfuzz** es un fuzzer. Nos va permitir reemplazar la palabra **FUZZ** por cualquier entrada que nosotros indiquemos. 

## 1. Enumeración de Directorios

Este es el uso más común. Buscamos carpetas dentro del servidor. 

```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hc 404 http://10.10.10.15/FUZZ
```

`-c` : Muestra la salida en colores (mucho más fácil de leer).

`-z file,[ruta]` : Define el "payload". Aquí le decimos que use un archivo como diccionario.

`--hc 404` : Crucial. Significa "Hide Code 404". Oculta todas las respuestas que no existan para que no se llene la pantalla de basura.

## 2. Filtrado 

En algunas ocasiones los servidores responden con respuestas como `200 OK` para todo, lo cual nos da falsos-negativos. **Para estos escenarios ocupamos**: 

`--hl [líneas]` : Oculta respuestas con un número específico de líneas.

`--hw [palabras]` : Oculta respuestas con un número específico de palabras.

`--hh [caracteres]` : Oculta respuestas por su tamaño en caracteres (muy útil cuando todas las páginas de error pesan lo mismo).

## 3. Doble Diccionario y Extensiones

Cuando queremos fuzzear varios archivos con posibles extensiones deberemos ocupar doble diccionario o lista. 

```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt -z list,php-txt-html --hc 404 http://10.10.10.15/FUZZ.FUZ2Z
```

`-z list,php-txt-html` : Crea un segundo "payload" (conjunto de datos).

`FUZ2Z` : Es el marcador de posición para el segundo payload.

Resultado: wfuzz probará cada palabra del diccionario con cada una de las extensiones que listaste (ej: index.php, index.txt, index.html).

**Nota:**  `-z list,php-txt-html` se puede reemplazar por un diccionario con: `-z file,/ruta/a/diccionario.txt`

## 4. Encoders

Lo que hace a **wfuzz** tan especial es que podemos usar **encoders** es decir transformar nuestro payload en diferentes cifrados, por ejemplo **base64, urlencode**.

De esta manera podemos ver los encoders:

```bash
wfuzz -e encoders
```

### URLencode

Si estás probando caracteres especiales en una URL, es mejor que vayan codificados:

```bash
wfuzz -z file,common.txt,urlencode http://10.10.10.15/FUZZ
```

### Base64

Si encontramos una página que pide una cookie de sesión o un parámetro que parece una cadena aleatoria, pero te das cuenta de que es Base64.

```bash
wfuzz -z file,users.txt,base64 http://10.10.10.15/profile.php?id=FUZZ
```

Si en tu diccionario tienes **admin**, wfuzz enviará **YWRtaW4=**.


