
---
- Tags: #Herramienta #Fuzzing #Enumeration 
- --
## Qué es?

**ffuf** (Fuzz Faster U Fool) Es una herramienta estandar que hoy en día forma parte de los **fuzzer** puros. **Recordemos que los fuzzer simplemente reemplazan la palabra *FUZZ* por una entrada de un diccionario** 

## 1. Descubrimiento de Directorios

La sintaxis más simple es:

```bash
ffuf -u http://10.10.10.10/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/
```

`-u` : La URL objetivo. El marcador FUZZ indica dónde se insertarán las palabras.

`-w` : El diccionario (wordlist). Puedes usar varios diccionarios separándolos por comas.

## 2. Filtrado 

Hay ocasiones en las que los servidores responden con **codigos 200** para todo, **eso llena de falsos/positivos**. **FFUF** nos permite filtraas los resultados. 

`-fc` (Filter Code): Filtra códigos HTTP específicos (ej: -fc 404,403).

 `-fs` (Filter Size): Filtra por el tamaño de la respuesta. Si ves que todos los errores 404 tienen un tamaño de 1542 bytes, usa -fs 1542.

`-fl` (Filter Lines): Filtra por el número de líneas en la respuesta.

`-fw` (Filter Words): Filtra por la cantidad de palabras.

## 3. Búsqueda de Archivos y Extensiones

Hay ocasiones en las que el archivo importante no es un directorio, sino un script. **Con el parámetro** 
`-e`
FFUF añade extensiones automáticamente a cada palabra de tu diccionario.

```bash
ffuf -u http://10.10.10.123/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt,.bak,.db
```

## 4. Descubrimiento de Vhosts

Recordemos, **muchas veces un servidor tiene varios sitios web alojados en la misma IP**.  Si únicamente escaneamos la IP veremos el sitio por defecto. 

```bash
ffuf -w /ruta/wordlist.txt -u http://10.10.10.10 -H "Host: FUZZ.objetivo.com"
```

**En esta técnica no cambiamos la URL, sino que cambiamos el Header *Host*** 

## 5. Fuzzing de Parámetros

Si llegamos a encontrar una página  `config.php` pero, está en blanco, a lo mejor y acepta un parametro:

`?file=`
`?page=`

 ```bash
 ffuf -u http://10.10.10.123/config.php?FUZZ=test -w /usr/share/wordlists/dirb/common.txt
 ```
 
## 6. Fuzzing Doble

Si queremos usar dos diccionarios, por ejemplo, un diccionario de rutas y otro de extensiones. 

```bash
ffuf -u http://objetivo.com/W1W2 -w nombres.txt:W1 -w mis_extensiones.txt:W2
```

`W1` : Se reemplaza por palabras como backup, admin, config.

`W2` : Se reemplaza por tus extensiones como .zip, .sql, .php.

**Resultado:** Probará todas las combinaciones (ej. backup.zip, backup.sql, admin.zip, etc.).

## 7. Brute Force

ffuf es tan rápido que puede usarse para ataques de fuerza bruta en formularios de inicio de sesión.

```bash
ffuf -u http://10.10.10.123/login.php -X POST -d "username=admin&password=FUZZ" -H "Content-Type: application/x-www-form-urlencoded" -ms 150-200
```

**Explicación:**

`-X POST` : Cambia el método de GET a POST.     

`-d` : Envía los datos del formulario.

`-ms` : (Match Size) Aquí buscarías una respuesta que tenga un tamaño diferente al de un "Login fallido".

