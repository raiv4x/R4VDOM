
---
- Tags: #Enumeration #Http #https 
- --- 

## En qué consiste?

En términos sencillos, la enumeración web es la fase de un ataque (o de una auditoría de seguridad) en la que se busca identificar y listar todos los componentes "ocultos" o no visibles a simple vista de un sitio web

**Imagina que quieres entrar a un edificio**: la enumeración es el proceso de caminar alrededor para encontrar todas las puertas traseras, ventanas mal cerradas, conductos de ventilación y saber qué empresas trabajan en cada piso.

## 1. Fuzzing de Directorios y Archivos

A menudo, los desarrolladores dejan carpetas expuestas (como /admin, /backup, o /config) que no están enlazadas en la página principal.**Nosotros como atacantes queremos descubrir esas carpeta expuestas**.

Para esto ocuparemos herramientas como:

**[[ffuf]]**
**[[wfuzz]]**
**[[gobuster]]**

**Es buena idea utilizar [[Dirbuster]]** como segunda opción a veces. 



