
---
- Tags: #Methodolgy #Checklist
- --- 

## 🛡️ Pentesting Methodology Checklist (eJPT)

    Resumen de Sesión Target: > Red: > Estado: 🔴 Recon | 🟠 Acceso | 🟢 Escala | 🔵 Pivot

### 📂 Fase 1: Organización (Setup)

    [ ] Estructura de Carpetas: mkdir -p {nmap,exploits,web,files,evidence}

    [ ] Archivo de Notas: Crear nota por cada IP identificada.

    [ ] Captura de Tráfico: tcpdump o wireshark (opcional).

### 🌐 Fase 2: Descubrimiento (Recon)

    [!info] Objetivo: Identificar qué máquinas están encendidas.

    [ ] ICMP Sweep: fping -asg <RED>/24 o nmap -sn <RED>/24

    [ ] ARP Scan (Local): arp-scan -l

    [ ] Netdiscover: netdiscover -r <RED>/24

### 🔍 Fase 3: Enumeración de Servicios

    "Enumeration is the key". No dejes ningún puerto sin investigar.

#### ⚙️ Escaneo General

    [ ] TCP Full Scan: nmap -p- -sV -sC -T4 -oN nmap/full <IP>

    [ ] UDP Top Scan: nmap -sU --top-ports 20 -oN nmap/udp <IP>

#### 🛠️ Servicios Específicos
##### FTP (21)

    [ ] Anon Login: nmap --script ftp-anon -p 21 <IP>

    [ ] Fuerza Bruta: hydra -C /usr/share/wordlists/metasploit/ftp_default_pass.txt ftp://<IP>

##### SSH (22)

    [ ] Banner: nc -vn <IP> 22

    [ ] User Enum: nmap --script ssh-auth-methods -p 22 <IP>

##### DNS (53)

    [ ] Zone Transfer: dig axfr @<IP> <domain> o host -l <domain> <IP>

    [ ] Recon: dnsrecon -d <domain> -n <IP>

##### WEB (80/443/8080)

    [ ] Tecnologías: whatweb <IP> o extensión Wappalyzer.

    [ ] Fuzzing Directorios: gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html

    [ ] Vulnerabilidades: nikto -h http://<IP>

##### SMB (139/445)

    [ ] General Enum: enum4linux -a <IP>

    [ ] Listar Shares: smbclient -L //<IP>/ -N

    [ ] Permisos: smbmap -H <IP>

##### SNMP (161)

    [ ] Comunidades: onesixtyone -c /usr/share/doc/onesixtyone/dict.txt <IP>

    [ ] Full Walk: snmpwalk -v2c -c public <IP>

### 💣 Fase 4: Explotación

    [ ] Búsqueda de Exploits: searchsploit <nombre_servicio> <version>

    [ ] Copia de Exploit: searchsploit -m <ID>

    [ ] Metasploit: msfconsole -> search, use, set options, exploit.

    [ ] Listener: nc -lvnp <PORT>

### 🔑 Fase 5: Post-Explotación

    Privilege Escalation Enumerar el sistema local para buscar debilidades.

#### 🐧 Linux

    [ ] Básicos: whoami, id, sudo -l, uname -a, cat /etc/crontab

    [ ] Archivos SUID: find / -perm -4000 -type f 2>/dev/null

    [ ] Scripts: linpeas.sh o lse.sh

#### 🪟 Windows

    [ ] Básicos: whoami /priv, systeminfo, net user, net localgroup

    [ ] Scripts: winpeas.exe o PowerUp.ps1

### 🚇 Fase 6: Pivotaje

    [ ] Rutas: ip route (Linux) o route print (Windows).

    [ ] Arp Table: arp -a

    [ ] Metasploit Pivot:

        run autoroute -s <RED_INTERNA>/24

        use auxiliary/server/socks_proxy

    [ ] Proxychains: Verificar /etc/proxychains4.conf.

##  **📝 Plantilla de Host (Target Template)**


Copia esta estructura para cada IP nueva.


| Puerto | Servicio | Versión | Comentarios |
| ------ | -------- | ------- | ----------- |
|        |          |         |             |
|        |          |         |             |
|        |          |         |             |
|        |          |         |             |

**Credenciales Encontradas:**

| USER | PASSWORD |
| ---- | -------- |
|      |          |

**Notas:**