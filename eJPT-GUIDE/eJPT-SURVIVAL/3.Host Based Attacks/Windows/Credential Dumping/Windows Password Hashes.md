
--- 
- Tags: #Credentials #Hashes #Windows #Post-exploitation 
--- 

# 1. Hashes

Para comenzar, tenemos que aprender donde se almacenan los hashes de Windows. **El sistema Windows, almacena los Hashes de manera local en la 'SAM'** (Security Accounts Manager) database. 

La **SAM** es la responsable de administrar usuarios y contraseñas en Windows. **Se necesitan privilegios elevados para poder administrar la SAM**. 

Los hashes NTLM se obtienen generalmente desde:

- **LSASS (memoria)**
	`LSASS es el proceso de Windows responsable de la autenticación, manejo de credenciales y políticas de seguridad. Es un objetivo crítico para atacantes y un punto clave de monitoreo para Blue Team.`
- SAM (usuarios locales)

## Qué son los hashes?

Es el proceso de convertir una contraseña en texto plano a otro valor. Hay diferentes algoritmos de Hashes. 

Los principales hashes en Windows son:

- **LM (LAN Manager)** 
- **NTLM**
- (NTLMv2 → mejora del NTLM)

# LM

Es un **hash viejo**, usado en Windows antiguos (pre-Vista).

Tiene **varios problemas graves**:

1. **Convierte la contraseña a MAYÚSCULAS**
2. **Máximo 14 caracteres**
3. Divide la contraseña en **2 bloques de 7 caracteres**
4. Usa **DES** (muy débil)
5. Si la contraseña es corta, **parte del hash queda vacío**


**Se puede:**

- Ataques **bruteforce MUY rápidos**
- Uso de **rainbow tables**

Por eso **Microsoft lo deshabilitó por defecto** en versiones modernas.

![[Pasted image 20260121192652.png]]


# NTLM (NTHash)

NTLM **no es solo un hash**, es un **protocolo de autenticación** de Windows.

- El **hash NTLM = “la contraseña”**
- Si tienes el hash → **eres el usuario**

![[Pasted image 20260121192954.png]]

