# 🚩 Ejercicio CTF: Cachopo 2 - TheHackersLabs  
### 👨‍💻 Autores: MY. AGUILAR - MY. CUBIDES

---

## 🎯 Objetivo

Aprender a realizar un análisis ofensivo de una máquina tipo CTF a través de técnicas de reconocimiento, enumeración, explotación y escalamiento de privilegios, utilizando herramientas disponibles en **Kali Linux** dentro de un entorno virtualizado con **VMware**.


## Reconocimiento

(images/a.png)


bash
Copiar
Editar
cd ~/Cachopo


bash
Copiar
Editar
nmap -sS -sV -T4 -Pn 192.168.184.2 -oN scans/scan_192.168.184.2.txt

> Comprueba si la máquina objetivo está activa.

### Conexión SSH

(images/b.png)

### Explotación Inicial

(images/c.png)

### Creación del diccionario y ataque de fuerza bruta con hydra

(images/d.png)





