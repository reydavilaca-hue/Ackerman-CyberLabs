# Obsession – Russoski Coaching DockerLab Walkthrough

**Researcher:** Ackerman  
**Platform:** DockerLabs  
**Difficulty:** Easy  
**Target IP:** 172.17.0.2  

---

## Description

Laboratorio enfocado en enumeración de servicios, acceso inicial mediante fuerza bruta y escalada de privilegios a través de una mala configuración en sudo.

---

## Tools Used

- :contentReference[oaicite:0]{index=0}
- :contentReference[oaicite:1]{index=1}
- :contentReference[oaicite:2]{index=2}
- FTP Client
- :contentReference[oaicite:3]{index=3}
- :contentReference[oaicite:4]{index=4}
- :contentReference[oaicite:5]{index=5}
- :contentReference[oaicite:6]{index=6}

---

## Reconnaissance

Se realizó un escaneo completo de puertos:

```bash
nmap -sV -sC -p- 172.17.0.2
