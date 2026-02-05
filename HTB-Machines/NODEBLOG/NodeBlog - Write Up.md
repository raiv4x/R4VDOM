
---
- Tags: #HTB #XXE #NoSQL #Easy #web 
- -- 

#### QUÉ SE EXPLOTA?

Se explotó una **inyección [[NoSQL]]** para evadir la autenticación, una **[[XXE]]** para leer archivos locales y una **[[deserialización]] insegura en NodeJS** que permitió ejecutar código. Luego se realizó **enumeración de MongoDB** para obtener información sensible.

#### Write UP

##### PRIMER FASE: Reconocimiento

Lo primero como siempre empezamos realizando **la primer fase de escaneos** que es para **descubrir todos los puertos disponibles** empezamos con un escaneo **[[syn scan]]**. 

Posteriormente seguimos con un escaneo un poco **más detallado*** para esto **utilizamos los scripts** por defecto de nmap. Además de eso **queremos descubrir los servicios por lo que ocupamos el script de servicios**.

**Descubrimos que hay 2 puertos abiertos** el puerto **22** y **5000** el puerto 22 es **ssh** y el puerto **5000** está corriendo un **http** 

Como buena costumbre revisamos el **launchpad** con el fin de comprobar codenames y **en caso de tener codenames distintos indicaría que posiblemente estemos frente a un docker**. 



##### SEGUNDA FASE: Explotación

**En la página web encontramos un panel de autenticación** y la página se ve muy mal montada. No hay mucho, más, no hay campos de registro, ni nada. **Probamos con admin:admin** y nos muestra que la contraseña es incorrecta. **Podriamos enumerar usuarios con fuerza bruta**. Intentamos con Inyección SQL. 

```bash
admin' or sleep(5)-- -
```

*Para investigar si hay  [[Blind SQLi]]*

Para poder usar más payloads a ver si alguno funciona, ocupamos [Payloadallthethings](https://github.com/swisskyrepo/PayloadsAllTheThings) y hay encontramos payloads de inyecciones [[NoSQLi]] así que también intentamos. **Recordemos que en burpsuite tenemos que cambiar el campo "content-type" en la petición por:**

```bash
application/json
```

Además de eso, para mandar
**Efectivamente logramos loggearnos** 

Una vez que logramos bypassear con la inyección. Se nos muestran algunos botones. **Podemos editar algunas cosas, subir archivos, borrar** . 

Proseguimos a investigar **que tipo de archivos se puede subir**. **Probamos subiendo un archivo .txt** Pero nos arroja error, y ademś nos muestra una estructura que tenemos que probar. 

**Básicamente tenemos que subir un [[XML]] por lo quepensamos en una [[XXE]] creamos un archivo [[XML]] y nuevamente en [Payloadallthethings](https://github.com/swisskyrepo/PayloadsAllTheThings)
buscamos XXE. 

