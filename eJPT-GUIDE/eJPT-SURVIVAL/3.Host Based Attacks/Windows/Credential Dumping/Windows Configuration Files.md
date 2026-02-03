
--- 
- Tags: #Hashes #Windows #Archivos #Post-exploitation 
- ---

# 1. Unattend.xml y Autounattend.xml 

**Unattend.xml** y **Autounattend.xml** son **archivos de configuración de instalación desatendida de Windows**.

Permiten **instalar Windows automáticamente**, sin intervención del usuario. **Generalmente en empresas o laboratorios**

## Diferencias:

| Archivo              | Cuándo se usa                            | Dónde se encuentra    |
| -------------------- | ---------------------------------------- | --------------------- |
| **Autounattend.xml** | Durante el **inicio de la instalación**  | USB / ISO / raíz      |
| **Unattend.xml**     | Durante la **configuración del sistema** | `C:\Windows\Panther\` |
## Importancia:

Porque **MUY A MENUDO** contienen:

- Contraseñas en **texto claro**
- Contraseñas **codificadas en Base64**
- Contraseñas de:
    
    - Administrador local
    - Cuentas de dominio
    - Servicios

## Rutas clave:

```bash
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Autounattend.xml
C:\Windows\System32\Sysprep\Unattend.xml
C:\Unattend.xml
C:\Autounattend.xml
```

## Cómo buscar?

```cmd
dir C:\ /s /b unattend.xml
dir C:\ /s /b autounattend.xml
```

