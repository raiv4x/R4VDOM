
--- 
- Tags: #Http #Https 
- ----
# Qué son? 

Indican **qué acción quiere hacer el cliente** sobre un recurso.

| Método      | Acción                 | Uso Común           | Riesgo en Pentesting             |
| ----------- | ---------------------- | ------------------- | -------------------------------- |
| **GET**     | Obtener datos          | Búsquedas, listados | Manipulación de parámetros, IDOR |
| **POST**    | Enviar datos           | Login, formularios  | SQLi, XSS, Auth Bypass           |
| **PUT**     | Subir / Reemplazar     | APIs, uploads       | File Upload → Web Shell          |
| **DELETE**  | Borrar recurso         | APIs                | Borrado no autorizado            |
| **PATCH**   | Modificar parcial      | APIs                | Cambio de campos sensibles       |
| **OPTIONS** | Ver métodos permitidos | Enumeración         | Descubrir superficie de ataque   |
| **HEAD**    | Igual que GET sin body | Fingerprinting      | Detección de recursos existentes |