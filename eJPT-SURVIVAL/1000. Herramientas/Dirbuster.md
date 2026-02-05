
---
- Tags: #Enumeration #Http #https 
- ---
## Que és?

Es una herramienta igualmente empleada para el descubrimiento de directorios. Su principal diferencia es que **Dirbuster** ocupa una **interfaz gráfica** muy intuitiva y amigable para el usuario. En ocasiones Dirbuster suele ser bastante preciso. 


## Cómo usar:

### 1. El Objetivo (Target)

En el campo **Target URL**, escribe la dirección completa que quieres analizar.

- Ejemplo: `http://192.168.1.50`


### 2. El Diccionario (Wordlist)

Tienes que decirle qué palabras usar. Haz clic en **Browse** y busca las listas que vienen por defecto (normalmente en `/usr/share/wordlists/dirbuster/`).

- **Recomendación:** Empieza con `directory-list-2.3-small.txt`. Es efectiva y no tarda una eternidad.


### 3. Configuración de búsqueda

- **Threads (Hilos):** Es la velocidad. Si le pones 10, hará 10 preguntas al mismo tiempo. No le pongas demasiado (máximo 20-30) o podrías tirar la página web o hacer que el servidor te bloquee por parecer un ataque.

- **File Extension:** Aquí le dices qué tipo de archivos buscar además de carpetas. Lo común es poner `php, html, txt`.


### 4. ¡A darle! (Start)