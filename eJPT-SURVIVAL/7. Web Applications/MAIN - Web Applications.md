

---
- Tags: #http #https #Web #Network 
- --- 
# Qué es ? 

Una aplicación web **es un software interactivo que se ejecuta directamente en el navegador web (como Chrome o Firefox) y se almacena en un servidor remoto**, eliminando la necesidad de instalación en el dispositivo del usuario

# 1. Componentes de una aplicación Web

Una aplicación web no es solo “una página”.  
Es un **ecosistema** de piezas que interactúan.

Podemos dividirlo así:

---

## 1. Cliente (Frontend)

Es **lo que ve y controla el usuario**.

### Tecnologías típicas

- **HTML** → estructura
- **CSS** → estilos
- **JavaScript** → lógica en el navegador
- Frameworks: React, Vue, Angular

### Qué se puede pentestear aquí

- **XSS (Cross-Site Scripting)**
- Manipulación de formularios
- Validaciones solo del lado cliente
- Cookies
- LocalStorage / SessionStorage

Regla de oro:  
**Nada del lado cliente es confiable.**  
Todo se puede modificar con DevTools.

---

## 2. Servidor (Backend)

Es **el cerebro**. Aquí se procesan las peticiones.

### Lenguajes comunes

- PHP
- Python (Django, Flask)
- Node.js
- Java
- Ruby

### Ataques típicos

- **SQL Injection**
- **Command Injection**
- **Path Traversal**
- **File Upload**
- **RCE**
- **SSRF**
- **LFI / RFI**

Aquí es donde vive la lógica real de seguridad.

---

## 3. Base de Datos

Donde se guardan los datos.

### Tipos

- **SQL**
    
    - MySQL
    - PostgreSQL
    - MSSQL
- **NoSQL**
    
    - MongoDB
    - Redis

### Ataques

- SQL Injection
- NoSQL Injection
- Enumeración de tablas
- Dump de credenciales

---

## 4. Servidor Web

Es quien **recibe las solicitudes HTTP** y las redirige al backend.

### Ejemplos

- Apache
- Nginx
- IIS

### Qué se revisa

- Versiones vulnerables
- Configuraciones malas
- Directorios expuestos
- Métodos HTTP habilitados
- Headers inseguros

---

## 5. Protocolo HTTP / HTTPS

Es **el idioma** entre cliente y servidor.

**INTERÉS MÁXIMO DEL HACKING**

### Cosas críticas

- Métodos: GET, POST, PUT, DELETE
- Headers
- Cookies
- Tokens
- Códigos de estado
- Parámetros

Herramientas clave:

- Burp Suite
- OWASP ZAP
- curl
- Postman

---

## 6. Autenticación y Sesiones

Dónde se rompe todo muchas veces.

### Vulnerabilidades comunes

- Session Hijacking
- Session Fixation
- JWT mal configurado
- Fuerza bruta
- Credenciales débiles
- Falta de expiración de sesión
---

## 7. APIs

Hoy en día son **uno de los puntos más explotables**.

### Riesgos

- IDOR
- Exposición de endpoints
- Tokens sin expiración
- Falta de rate limiting
- Autorización incorrecta


# 2. Procesamientos

## Client-Side

Es todo lo que ocurre **en el navegador del usuario**.
**Tecnologías:**

- HTML
- CSS
- JavaScript
- Frameworks JS

El navegador **ejecuta código** antes de enviar datos al servidor.

---
### ¿Qué se procesa aquí?

#### 1. Validaciones de Formularios

Ejemplo:

- Campo solo números
- Email obligatorio
- Longitud mínima

Problema:  
Puedes **desactivarlo** o **editar la request**.

#### 2. Ocultar/Mostrar Elementos

Botones de admin ocultos con JS.

Pentester:

- DevTools → quitar `display:none`
- Elemento vuelve a aparecer

#### 3. Manipulación de Datos

JavaScript puede:

- Cambiar valores
- Construir JSON
- Generar tokens temporales

Pero todo esto **se puede modificar**.

---

#### 4. Almacenamiento Local

- Cookies
- LocalStorage
- SessionStorage

**Riesgo:**

- Tokens expuestos
- Datos sensibles visibles

## Server-Side

Aquí vive **la seguridad real**.

Lenguajes:

- PHP
- Python
- Java
- Node
- Ruby

El usuario **no ve este código**.  
Solo ve resultados.

---

### ¿Qué se procesa aquí?

#### 1. Validaciones Reales

- Tipos de datos
    
- Longitud
    
- Sanitización
    

Si falla → SQLi, RCE, etc.

---

#### 2. Autenticación

- Login
    
- Hash de contraseñas
    
- Sesiones
    
- Tokens
    

---

#### 3. Autorización

Permisos de usuario.

Aquí nacen:

- IDOR
    
- Broken Access Control
    

---

#### 4. Interacción con Base de Datos

- Queries
    
- Inserts
    
- Updates
    

Si no se limpian datos → SQL Injection.

---

#### 5. Subida de Archivos

- Validar extensión
    
- MIME type
    
- Directorio
    

Si falla → Web Shell.

---

#### 6. Ejecución de Comandos

- Llamadas al sistema
    
- Scripts
    

Si no se filtra input → Command Injection.

---
# 3. Flujo de Datos y Comunicación en Web

Es **el camino que sigue la información** desde que el usuario envía algo hasta que el servidor responde.

## Diagrama Mental

```bash
Usuario
  ↓
Navegador
  ↓
HTTP Request  →  Servidor Web  →  Backend  →  Base de Datos
  ↑                                                     ↓
  └────────────── HTTP Response  ←────────────────────┘

```

  ![[Pasted image 20260131211509.png]]