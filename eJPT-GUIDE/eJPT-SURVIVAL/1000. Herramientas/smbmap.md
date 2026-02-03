
---
- Tags: #Herramienta #SMB 
- -- 
## Qué es? 

**SMBmap** es la herramienta que nos va a permitir enumerar los permisos dentro de cada cada share así como descargar o subir recursos. 

## Cómo usar? 

### Enumeración Anónima

```bash
smbmap -H <IP>
```

Si llegamos a ver `READ ONLY` o `READ, WRITE` en algún recurso que no sea el estándar `IPC$`, hemos encontrado una brecha.

### Enumeración con credenciales

En caso de que ya tengamos algunas credenciales

```bash
smbmap -u <usuario> -p <passwd> -H <IP>
```

### Recursividad y busqueda

Podemos listar todos los recursos de los shares y buscar archivos específicos.

- **Recursividad:**
  
```bash
smbmap -H <IP> -r
```

Lista todos los archivos de todos los shares legibles

- **Busqueda con Regex**:
  
  ```bash
  smbmap -H <IP> -R <Share> -A "(config|password|backup)"
  ```

**Traducción:** "Busca recursivamente en este share cualquier archivo que tenga en su nombre las palabras 'config', 'password' o 'backup'".

### Post-Explotación

Si llegamos a encontrar algún share con permisos `WRITE` **smbmap** se convierte en una herramienta de ataque.

- **Descargar un archivo**
  
  ```bash
  smbmap -H <IP> --download "C$\Users\Admin\Desktop\pass.txt"
  ```

- **Subir archivos**
  
  ```bash
  smbmap -H <IP> --upload ruta/archivo/ "c$\temp\shell.exe"
  ```

- **Ejecución de comandos (Si tenemos admin)**
  
  ```bash
  smbmap -H <IP> -x "whoami"
  ```


