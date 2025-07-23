# 🚩 Ejercicio CTF: Cachopo 2 - TheHackersLabs  
### 👨‍💻 Autores: MY. AGUILAR - MY. CUBIDES

---

## 🎯 Objetivo

Aprender a realizar un análisis ofensivo de una máquina tipo CTF a través de técnicas de reconocimiento, enumeración, explotación y escalamiento de privilegios, utilizando herramientas disponibles en **Kali Linux** dentro de un entorno virtualizado con **VMware**.

> ⚠️ **Este ejercicio tiene fines exclusivamente educativos. No debe ser utilizado en sistemas sin autorización.**

---

## Reconocimiento

![crea la carpeta antes Ejecuta este comando para crear la estructura de carpetas necesarias](IMAGES/a.png


bash
Copiar
Editar
cd ~/Cachopo
Y ahora sí puedes correr el escaneo sin problemas:

bash
Copiar
Editar
nmap -sS -sV -T4 -Pn 192.168.184.2 -oN scans/scan_192.168.184.2.txt

> Comprueba si la máquina objetivo está activa.

### Conexión SSH

![](IMAGES/b.png

### Explotación Inicial

![uso de herramienta Hydra](IMAGES/c.png

### Creación del diccionario y ataque de fuerza bruta con hydra

![](IMAGES/d.png

### 

![](IMAGES/d.png

![](IMAGES/d.png

