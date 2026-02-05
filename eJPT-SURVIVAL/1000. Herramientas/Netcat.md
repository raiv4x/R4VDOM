
---
- Tags: #Herraienta #Enumeration #SMTP #TCP
- -- 
# Qué es?

**Netcat (nc)** es una navaja suiza de redes.  
Permite **leer y escribir datos sobre TCP y UDP**.

En pentesting lo usamos para:
- Conectarnos a servicios
- Levantar listeners
- Transferir archivos
- Reverse shells / bind shells
- 
Netcat **mueve bytes**.
Linux decide:

- qué entra → `>`
- qué sale → `<`
- cómo fluye → `|`

# Usos

## 1. Modo Cliente

Es el modo por default de **netcat**

```bash
nc 10.10.10.10 22
```
_Nos conectamos a puerto 22_

Usos:

- Ver si un puerto responde
- Interactuar con servicios (HTTP, SMTP, FTP)
- Mandar payloads manuales

**Nota:** La flag `-e` es de suma importancia. 
Viene de `execute` por tanto al utilizarse se ejecuta cualquier binario (comando) que se le indique.  _Muy utilizado en **[[Bind shells]]**_

## 2. Modo Escucha (Listen)

**Netcat** se queda esperando conexiones entrantes. 

```bash
nc -lvnp 443
```

_Muy usado en **[[Reverse shells]]**_

## 3. Transferencia de Archivos

Desde la máquina receptora:

```bash
nc -lvp 4444 > archivo.txt
```

Desde la máquina emisora:

```bash
nc 10.10.10.10 4444 < archivo.txt
```



