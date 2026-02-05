
# Lab Environment

In this lab environment, you will be provided with GUI access to a Kali Linux machine. Two machines are accessible at **http://target1.ine.local** and **http://target2.ine.local**.

**Objective:** Perform system/host-based attacks on the target and capture all the flags hidden within the environment.

**Useful files:**

```
/usr/share/metasploit-framework/data/wordlists/common_users.txt, 
/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt,
/usr/share/webshells/asp/webshell.asp
```

**Flags to Capture:**

- **Flag 1**: User 'bob' might not have chosen a strong password. Try common passwords to gain access to the server where the flag is located. (target1.ine.local)
- **Flag 2**: Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)
- **Flag 3**: By attempting to guess SMB user credentials, you may uncover important information that could lead you to the next flag. (target2.ine.local)
- **Flag 4**: The Desktop directory might have what you're looking for. Enumerate its contents. (target2.ine.local)

# Tools

The best tools for this lab are:

- Nmap
- Hydra
- Cadaver
- Metasploit Framework