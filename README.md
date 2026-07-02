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

Se realizó un escaneo completo de puertos:

```bash
nmap -sV -sC -p- 172.17.0.2
