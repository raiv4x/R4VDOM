
---
- Tags: #INE #Lab #Enumeration 
---

# Lab Environment

A website is accessible at **http://target.ine.local**. Perform reconnaissance and capture the following flags.

- **Flag 1:** This tells search engines what to and what not to avoid.
- **Flag 2:** What website is running on the target, and what is its version?
- **Flag 3:** Directory browsing might reveal where files are stored.
- **Flag 4:** An overlooked backup file in the webroot can be problematic if it reveals sensitive configuration details.
- **Flag 5:** Certain files may reveal something interesting when mirrored.

# Tools

- Firefox
- Curl
- HTTrack


# Write Up

Empezamos haciendo un escaneo con nmap para descubrir todos los puertos abiertos. 

```bash
nmap -sS -p- -Pn -n --open --min-rate 5000 -vv -oN tcp_scan target.ine.local
```

![[Pasted image 20260114103316.png]]

Una vez que ya descubrimos un puerto **pasamos a determinar los servicios y versiones junto con otros scripts**

```bash
nmap -sCV -p 80 -T 4 target.ine.local -oN port_scan
```

![[Pasted image 20260114103749.png]]

**Encontramos la FLAG 2**

```bash
nmap --script http-enum  -p 80 -T 4 target.ine.local 
```

![[Pasted image 20260114103820.png]]

**Existe el robots.txt** así que checamos que hay

![[Pasted image 20260114103935.png]]

Encontramos la **FLAG1**

**PASAMOS A ENUMERAR DIRECTORIOS Y ARCHIVOS**

```bash
gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -u http://target.ine.local/ -t 200 -x php,html,bak,tar,php.bak,gz,zip 2>/dev/
```

![[Pasted image 20260114104127.png]]

**El archivo config.bak** se ve interesante puede ser un backup anterior. 

```bash
curl http://target.ine.local/wp-config.bak
```

![[Pasted image 20260114104512.png]]

Buscando en todo el output encontramos la **FLAG4**

Proseguimos a derscubrir más directorios. 

**En wp-content** encontramos una carpeta llamada **uploads**

![[Pasted image 20260114105157.png]]

```bash
curl http://target.ine.local/wp-content/uploads/ | grep 'flag'
```

![[Pasted image 20260114105338.png]]

```bash
curl http://target.ine.local/wp-content/uploads/flag.txt
```

![[Pasted image 20260114105617.png]]

**Encontramos la FLAG3**

Ahora **Proseguimos a buscar la FLAG5** de acuerdo a las pistas tenemos que hacer mirror al sitio, para eso ocupamos **[[httrack]]**

```bash
httrack http://target.ine.local -%v -o target.ine.local
```

![[Pasted image 20260114110302.png]]

Se descarga el sitio, nos metemos a la carpeta y buscamos

```
grep -r 'FLAG'
```

![[Pasted image 20260114110332.png]]

