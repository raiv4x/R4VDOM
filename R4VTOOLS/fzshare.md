
---
- Tags: #Bashscript #Herramienta #SMB #Fuzzing
- -- 

#Script destinado a fuzzear rutas de shares con acceso NULO ej: \\10.10.10.10\FUZZ


```bash
#!/bin/bash

IP=$1
WORDLIST=$2

# Colores
VERDE="\e[32m"
CYAN="\e[36m"
RESET="\e[0m"
BORRAR_LINEA="\e[2K\r"

# Comprobaciones iniciales
if [ -z "$IP" ] || [ -z "$WORDLIST" ]; then
    echo -e "${CYAN}Uso:${RESET} $0 <IP> <WORDLIST>"
    exit 1
fi

if [ ! -f "$WORDLIST" ]; then 
    echo -e "No existe el diccionario usado."
    exit 1
fi

echo -e "\n${CYAN}[*] Iniciando escaneo de SMB en $IP...${RESET}\n"

# Inicio del fuzzing
while read -r share; do
    # Línea de estado dinámica (se sobrescribe a sí misma)
    echo -ne "${BORRAR_LINEA}[?] Probando: $share"
    
    # Probamos la ruta (Null Session)
    smbclient "//$IP/$share" -N -c "ls" &>/dev/null
    
    # Verificamos éxito
    if [ $? -eq 0 ]; then
        echo -e "${BORRAR_LINEA}${VERDE}[+] ACCESO EXITOSO: //$IP/$share${RESET}"
    fi
    
done < "$WORDLIST"

echo -e "\n\n${CYAN}[*] Escaneo finalizado.${RESET}"
```



