---

---

---
 - Tags: #Herramienta #Bashscript #Post-exploitation #Enumeration 
---

```bash
old_proc=$(ps -eo cmd --no-headers | sort)

while true; do
    sleep 10
    new_proc=$(ps -eo cmd --no-headers | sort)

    echo "=== Nuevos procesos ==="
    diff <(echo "$old_proc") <(echo "$new_proc") | grep "^>"

    old_proc="$new_proc"
done
```

**Sirve para detectar procesos que se ejecutan en intervalos de tiempo**.