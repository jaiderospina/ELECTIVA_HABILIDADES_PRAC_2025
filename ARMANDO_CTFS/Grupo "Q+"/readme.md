<h1 align="center"><img width="768" height="456" alt="image" src="https://github.com/user-attachments/assets/e7be799d-2c6a-4298-8687-c1e6b3d89c37" /></h1> 
<h1 align="center">:rotating_light::bulb:SIMULACIÓN DE ATAQUE FTP CTF EN KALI LINUX:bulb::rotating_light:</h1> 

# :eight_spoked_asterisk: **Rubén Contreras Caballero** :anchor:
# :eight_spoked_asterisk: **Johan Martínez Rojas** :anchor:
# :eight_spoked_asterisk: **Mauricio Gómez Rodríguez** :tent:

# Introducción

Este informe describe detalladamente el proceso de creación, configuración y explotación de un servicio FTP vulnerable utilizando tecnologías como Docker, Kali Linux, Nmap e Hydra, en el marco de una práctica de ciberseguridad ofensiva. El objetivo fue simular un entorno real de captura de bandera (CTF) en el cual se despliega una imagen personalizada de Docker que contiene un servicio FTP con credenciales conocidas, para posteriormente realizar un escaneo de puertos y un ataque de fuerza bruta dirigido a obtener el acceso al servicio FTP.

A lo largo del documento, se detallan los pasos realizados, se muestran las evidencias visuales correspondientes, y se explican los comandos utilizados, con el fin de documentar claramente cada fase del procedimiento. Finalmente, se presentan las conclusiones derivadas de la práctica, resaltando los aprendizajes obtenidos y la utilidad del ejercicio como entrenamiento práctico en ciberseguridad.

# 📝 Descripción del Proyecto

En este laboratorio, se detalla el proceso completo para:

- Crear una imagen Docker personalizada con un servidor FTP (vsftpd) deliberadamente mal configurado.

- Publicar y desplegar dicha imagen como un contenedor.

- Realizar un escaneo de puertos para verificar que el servicio está expuesto.

- Ejecutar un ataque de fuerza bruta por diccionario para obtener las credenciales de acceso.

Este ejercicio es fundamental para comprender el ciclo de vida de un ataque en un entorno seguro y legal.

# 🚀 Guía Paso a Paso

A continuación se detalla cada fase del ejercicio, desde la preparación del entorno hasta la explotación final.

## Paso 1: Crear directorio y archivo Dockerfile
Se intentó acceder al directorio `ftp_ctf`, pero luego se identificó que el nombre correcto era `ftp_cft`. Dentro de este directorio se encuentra el archivo `Dockerfile` con la configuración del entorno FTP.

Comando utilizado:
```dockerfile
cd ftp_cft
```
<h1 align="center"><img width="900" height="431" alt="image" src="https://github.com/user-attachments/assets/5ace74de-4edc-4952-8d13-ac7a2f36ec71" /></h1> 
Figura 1. Acceso al directorio de trabajo.

## Paso 2: Construcción de la imagen Docker
Se ejecutó el comando para construir la imagen Docker a partir del archivo `Dockerfile`. La imagen se etiquetó como `retoftp`. El Dockerfile crea un servicio vsftpd con el usuario 'JOHAN' y contraseña '1234'.

Comando utilizado:
```dockerfile
sudo docker build -t retoftp .
```
<h1 align="center"><img width="900" height="431" alt="image" src="https://github.com/user-attachments/assets/e96d3063-ade5-400b-ac02-2fe1d5dedec7" /></h1> 
<h1 align="center"><img width="900" height="431" alt="image" src="https://github.com/user-attachments/assets/5e174813-014a-4751-8db9-cda5f105c0ed" /></h1> 
Figura 2. Proceso de construcción de imagen Docker.

## Paso 3: Verificación de la imagen creada

Comando utilizado:
```dockerfile
sudo docker images
```
<h1 align="center"><img width="825" height="395" alt="image" src="https://github.com/user-attachments/assets/66b8c0d9-4e65-4f1b-ad96-034b127d6b48" /></h1> 
Figura 3. Verificación de la imagen creada

## Paso 4: Inicio de sesión en Docker Hub
Se inició sesión correctamente en Docker Hub con el usuario `martinez5125` luego de varios intentos fallidos.

Comando utilizado:
```dockerfile
sudo docker login
```
<h1 align="center"><img width="900" height="431" alt="image" src="https://github.com/user-attachments/assets/361e0dcb-d231-4b75-b26b-11c1622f4ea6" /></h1>
Figura 4. Inicio de sesión exitoso en Docker Hub.

## Paso 5: Etiquetado y subida de la imagen
Se etiquetó la imagen local con el nuevo nombre `martinez5125/retoftp:1.0` y se subió al repositorio remoto.

Comandos utilizados:
```dockerfile
sudo docker tag retoftp martinez5125/retoftp:1.0
sudo docker push martinez5125/retoftp:1.0
```
<h1 align="center"><img width="825" height="395" alt="image" src="https://github.com/user-attachments/assets/e8fed1b7-4d37-4b14-8dfd-a77d193e1ffa" /></h1>
Figura 5. Etiquetado y subida de imagen al repositorio

## Paso 6: Ejecución del contenedor FTP
Se ejecutó el contenedor utilizando el puerto 2222 redirigido al 21 del contenedor. Se solucionó un conflicto de nombres eliminando un contenedor anterior.

Comandos utilizados:
```dockerfile
sudo docker stop retoftp
sudo docker rm retoftp
sudo docker run -d -p 2222:21 --name retoftp martinez5125/retoftp:1.0
```
<h1 align="center"><img width="825" height="395" alt="image" src="https://github.com/user-attachments/assets/d9c5a373-9c81-4e49-b08b-75d83b3e9a3e" /></h1>
Figura 6. Ejecución del contenedor

## Paso 7: Verificación de puertos con Nmap
Se verificó con `nmap` que el puerto 2222 estaba correctamente abierto y escuchando conexiones FTP.

Comandos utilizados:
```dockerfile
nmap -p 2222 localhost
```
<h1 align="center"><img width="825" height="395" alt="image" src="https://github.com/user-attachments/assets/6ce9bab0-fc03-467c-8db3-1b446549f5b1" /></h1>
Figura 7. Escaneo de puertos con Nmap

## Paso 8: Ataque con Hydra (pruebas y éxito)
Se utilizó Hydra para realizar un ataque de fuerza bruta contra el servicio FTP. Después de varios intentos fallidos, se encontró correctamente la combinación: `usuario: JOHAN` y `contraseña: 1234`.

Comando final exitoso:
```dockerfile
hydra -l JOHAN -P diccionario.txt ftp://localhost:2222 -t 4
```
<h1 align="center"><img width="825" height="395" alt="image" src="https://github.com/user-attachments/assets/6accc444-c0e8-49f8-8c2e-ea036f58231e" /></h1>
Figura 8. Ataque de diccionario con Hydra

# 🧠 Conclusiones y Aprendizajes
Esta práctica permitió aplicar herramientas fundamentales en ejercicios de hacking ético, destacando la importancia de dominar conceptos como la construcción de imágenes Docker, la administración de contenedores, el escaneo de servicios y la explotación mediante ataques de diccionario.

Entre los aprendizajes obtenidos se destaca:

- La capacidad de personalizar un entorno vulnerable para pruebas controladas.

- La identificación y validación de servicios activos a través de Nmap.

- El uso de Hydra como herramienta efectiva para comprometer credenciales débiles en servicios como FTP.


