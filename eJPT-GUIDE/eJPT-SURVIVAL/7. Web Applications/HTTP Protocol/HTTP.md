
--- 
- Tags: #Http #https #Web #Network 
- --- 
# Qué es?

**HTTP (HyperText Transfer Protocol)**  
Es el **idioma** que usa el navegador para hablar con el servidor.

Todo lo que ves en una web son:

- **Requests** (peticiones)
- **Responses** (respuestas)

## HTTP 1.1 Y HTTP 1.0

### HTTP 1.0

#### Características

- **Una conexión por petición**
- Cada request abre TCP y luego lo cierra.
- **No existe `Host` header obligatorio.**
- No hay persistencia.
- Muy básico.

```bash
GET /index.html HTTP/1.0
```

El servidor responde y **cierra conexión**.

---

### HTTP 1.1

Es el estándar moderno.
#### Características

- **Conexión persistente (Keep-Alive)**
- **`Host:` obligatorio**
- Transfer-Encoding
- Chunked encoding
- Pipeline
- Mejor rendimiento

```bash
GET /index.html HTTP/1.1
Host: target.com
Connection: keep-alive
```

Posibles impactos:

- Password reset poisoning
- SSRF
- Cache poisoning
- Open redirect
- Bypass de WAF

---

# 1. Estructura de HTTP

HTTP funciona en **Request → Response**

---
## 1. HTTP Requests (Peticiónes)

Es lo que envía el cliente.
### Estructura básica

```bash
METODO /ruta HTTP/1.1
Host: ejemplo.com
Header: valor
Cookie: session=123

cuerpo(opcional)
```

#### 1. Request Line (Línea de Petición)

Es **la primera línea**.  
Define **qué se quiere hacer y dónde**. Utiliza los **[[Métodos HTTP]]**

Formato:

`METODO /ruta?param=valor HTTP/1.1`

Ejemplo:

`POST /login.php HTTP/1.1`

Contiene:

- **[[Métodos HTTP]]** → GET, POST, PUT, DELETE…
- **Ruta/Endpoint** → `/login.php`
- **Versión HTTP** → `HTTP/1.1`

**Pentesting aquí:**

- Cambiar GET ↔ POST
- Modificar ruta
- Probar endpoints ocultos
- Path Traversal

####  2. Request Headers (Líneas de Headers)

No son los datos principales, pero **controlan cómo el servidor interpreta la petición**.

**Formato**:

```bash
Header: Valor
```

---
##### Headers Clave

**Host** → Dominio destino

- Virtual hosts, host header injection


**User-Agent** → Navegador/dispositivo

- Bypass de filtros básicos


**Cookie** → Sesión del usuario

- Session hijacking / fixation


**Authorization** → Tokens / credenciales

- Reuso o manipulación de JWT


**Content-Type** → Formato del body

- Cambiar a JSON / bypass validaciones


**Referer** → De dónde viene la petición

- Bypass de controles débiles


**Origin** → Origen para CORS

- Detectar CORS mal configurado