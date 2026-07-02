# Obsession – Attack Path Analysis: FTP, SSH and Privilege Escalation

**Researcher:** Ackerman  
**Platform:** DockerLabs  
**Difficulty:** Easy  
**Target IP:** 172.17.0.2  

---

## Description

Laboratorio enfocado en enumeración de servicios, acceso inicial mediante fuerza bruta y escalada de privilegios a través de una mala configuración en sudo.

---

# Tools Used

- *Nmap* (Port scanning & service enumeration)
- Gobuster (Directory brute forcing)
- Hydra (Credential brute force)
- FTP Client (File enumeration)
- OpenSSH (Remote access)
- ExifTool (Metadata analysis)
- Vim (Privilege escalation)
- SecLists (Wordlists)

---

## Reconnaissance

Se realizó un escaneo completo de puertos:

```bash
nmap -sV -sC -p- 172.17.0.2
