
--- 
- Tags: #Lab #INE #Enumeration 
- --- 

# Lab Environment

A Linux machine is accessible at **target.ine.local**. Identify the services running on the machine and capture the flags. The flag is an md5 hash format.

- **Flag 1:** There is a samba share that allows anonymous access. Wonder what's in there!
- **Flag 2:** One of the samba users have a bad password. Their private share with the same name as their username is at risk!
- **Flag 3:** Follow the hint given in the previous flag to uncover this one.
- **Flag 4:** This is a warning meant to deter unauthorized users from logging in.

**Note: The wordlists located in the following directory will be useful:**

- /root/Desktop/wordlists

# Tools

- Nmap
- Metasploit
- Hydra
- enum4linux
- smbclient
- smbmap


# Write Up

## 1. Organización

Empezamos creando una ruta donde vamos a empezar a trabajar.

![[Pasted image 20260109174600.png]]

```bash
mkdir LAB
```

Posteriormente crearemos nuestro arbol de trabajo con:

```bash
mkdir -p target.ine.local/{nmap,content,exploits,evidence}
```

![[Pasted image 20260109174820.png]]

## 2. Reconocimiento

Empezaremos utilizando nmap para **Descubrir todos los puertos TCP abiertos. 

```bash
nmap -sS -p- --open --min-rate 5000 -T 4 -Pn -n -vv -oG tcp_scan target.ine.local 
```

![[Pasted image 20260109175024.png]]

**Con grep y algunas otras herramientas extraemos los puertos**:

```bash
cat tcp_scan | grep -oP '\d+(?=/open)' | xargs | tr ' ' ','
```

![[Pasted image 20260109175247.png]]

Ahora pasamos a realizar un segundo escaneo. **Nuestro objetivo ahora será obtener más detalles sobre los puertos encontrados**. 

![[Pasted image 20260109175650.png]]

Podemos identificar **SSH** (No es vulnerable a enumeración). **SMB** y tampoco es vulnerable. 
**Algo raro**: **FTP** está en otro puerto. 

**En cuanto al escano UDP** : Solo encontramos **Puerto 137** el cual corresponde a NetBios de **SMB**. 

![[Pasted image 20260109180338.png]]

**Proseguimos a utilizar**: **[[enum4linux]]**.

Ya que el servicio **SMB** estaba corriendo decidimos ver si encontramos algo interesante:

```bash
enum4linux -U target.ine.local
```

![[Pasted image 20260109180933.png]]

**Proseguimos a buscar SHARES**

![[Pasted image 20260109181112.png]]

Solamente para confirmar utilizamos [[smbclient]]

![[Pasted image 20260109181017.png]]

Como no encontramos ni un share, **pasamos a fuzzear shares**, para esto utilizamos **[[fzshare]]**

**Hay que recordar la nota del incio del LAB**

![[Pasted image 20260109184305.png]]

**Además** , como ya teníamos algunos usuarios válidos **utilizando el modulo de smb_login e hicimos un ataque de fuerza bruta

![[Pasted image 20260109184419.png]]

**Ya teniamos credenciales válidas por lo que intentamos entrar a través de SSH con las credenciales que ya tenemos**

![[Pasted image 20260109184734.png]]

**Encontramos una bandera** : FLAG4 

Ahora como anteriormente ya habíamos encontrado un share **/pubfiles** nos conectamos a el.

```bash
smbclient //target.ine.local/pubfiles
```

**Como no pide contraseña** nos conectamos, listamos el contenido y descargamos la flag:
![[Pasted image 20260109184945.png]]

```bash
get flag1.txt
```

**Siguiendo la FLAG2** Nos dice que el share oculto es igual al nombre del usuario con una contraseña mala.

```bash
smbclient //target.ine.local/josh -U josh
```

Como ya tenemos la contraseña la proporcionamos: **purple**

![[Pasted image 20260109185353.png]]

Listamos el contenido y estaba la **FLAG2** 

![[Pasted image 20260109185445.png]]

Para la última bandera nos dice que hay una pista en la bandera anterior. **Tiene que ver con el servicio FTP que encontramos**

Como ya tenemos una lista con posibles usuarios, únicamente vamos a intentar con fuerza bruta a ver si podemos encontrar algo. **Además vamos a checar el banner** 

Para checar el banner usamos **ftp_version** de metasploit

![[Pasted image 20260109190235.png]]

**Encontramos más posibles usuarios**

- ashley
- alice
- amanda

Además dice que tienen contraseñas debiles. 

**Con [[hydra]]** intenamos un ataque de fuerza bruta a **FTP**

```bash
hydra -L users_ftp -P /root/Desktop/wordlists/unix_passwords.txt ftp://target.ine.local:5554

```

![[Pasted image 20260109190537.png]]

Encontramos la contraseña, **Nos conectamos y encontamos la flag**

```
ftp alice@target.ine.local -P 5554
```

```bash
dir - Listamos
Binary - Entramos en modo binario para transferencia
get flag3.txt
```

![[Pasted image 20260109190822.png]]

**Listo** Tenemos todas las flags.

![[Pasted image 20260109190921.png]]

