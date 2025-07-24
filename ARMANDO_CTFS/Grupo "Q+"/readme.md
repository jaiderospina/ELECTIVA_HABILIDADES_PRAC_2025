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

## Fase 1: Creación y Despliegue del Contenedor Vulnerable
En esta fase, construiremos nuestra propia imagen de Docker, la subiremos a un registro público (Docker Hub) y la ejecutaremos localmente.

Acceso al Directorio del Proyecto
Nos ubicamos en la carpeta que contiene el Dockerfile y los archivos necesarios.

bash
cd ftp_ctf
Figura 1: Acceso al directorio ftp_ctf.

<!-- -->
Construcción de la Imagen Docker
Creamos la imagen a partir de nuestro Dockerfile, asignándole el nombre retoftp.

bash
sudo docker build -t retoftp .
Figura 2: Salida del comando de construcción de la imagen.

<!-- ![Construcción de imagen](assets/fign** Listamos las imágenes locales para confirmar que `retoftp` fue creada correctamente. ```bash sudo docker images ``` *Figura 3: Verificación de la imagen creada en el listado de Docker.* <!-- -->
Publicación en Docker Hub
Para compartir o desplegar la imagen en otros sistemas, la subimos a un registro. Primero, iniciamos sesión, luego la etiquetamos y finalmente la publicamos.

bash
# Paso 4: Inicio de sesión
sudo docker login

# Paso 5: Etiquetado (tagging) y subida (push)
sudo docker tag retoftp martinez5125/retoftp:1.0
sudo docker push martinez5125/retoftp:1.0
Figuras 4 y 5: Proceso de login, etiquetado y subida de la imagen.

<!-- --> <!-- -->
Ejecución del Contenedor
Ejecutamos la imagen, mapeando el puerto 21 del contenedor al puerto 2222 de nuestra máquina anfitriona (Kali).

bash
sudo docker run -d -p 2222:21 --name retoftp martinez5125/retoftp:1.0
Figura 6: Ejecución del comando para iniciar el contenedor.

<!-- -->
## Fase 2: Auditoría y Ataque al Servicio FTP
Con el contenedor activo, procedemos a realizar la auditoría de seguridad.

Verificación del Contenedor y Puerto
Confirmamos que el contenedor está corriendo y qué puerto está expuesto usando docker ps y nmap.

bash
# Verificar contenedor activo
sudo docker ps

# Escanear el puerto expuesto en la máquina local
nmap -p 2222 localhost
Figura 7: Escaneo con Nmap que confirma el puerto 2222 abierto.

<!-- -->
Ataque de Diccionario con Hydra
Lanzamos el ataque de fuerza bruta. Hydra intentará autenticarse con el usuario JOHAN y cada una de las contraseñas listadas en diccionario.txt.

bash
hydra -l JOHAN -P diccionario.txt ftp://localhost:2222 -t 4
Figura 8: Hydra encuentra con éxito la contraseña correcta.

<!-- -->
🐳 Análisis del Dockerfile
El Dockerfile es la receta para construir nuestro entorno vulnerable. A continuación, se desglosan sus directivas clave:

text
# Se utiliza una imagen base de Ubuntu
FROM ubuntu:latest

# Se actualizan los repositorios y se instala el servidor FTP (vsftpd) y sudo
RUN apt-get update && \
    apt-get install -y vsftpd sudo && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Se crea un nuevo usuario 'JOHAN' y se le asigna la contraseña '1234'
RUN useradd -m -d /home/JOHAN -s /bin/bash -G sudo JOHAN && \
    echo "JOHAN:1234" | chpasswd

# Se crea y configura el archivo de configuración de vsftpd
# Se habilitan directivas que permiten la escritura y el acceso del usuario
RUN printf "listen=YES\nlisten_ipv6=NO\nlocal_enable=YES\nwrite_enable=YES\nchroot_local_user=YES\nallow_writeable_chroot=YES\npasv_enable=YES\npasv_min_port=30000\npasv_max_port=30009\n" > /etc/vsftpd.conf

# Se exponen los puertos para la conexión FTP
EXPOSE 21 30000-30009

# Comando que se ejecutará al iniciar el contenedor para levantar el servicio FTP
CMD ["/usr/sbin/vsftpd", "/etc/vsftpd.conf"]
🔧 Resumen de Herramientas y Comandos
Herramienta / Comando	Propósito en el Ejercicio	Fase
docker build	Construye una imagen de Docker a partir de un Dockerfile.	Preparación
docker images	Lista todas las imágenes de Docker almacenadas localmente.	Preparación
docker push	Sube una imagen a un registro remoto como Docker Hub.	Despliegue
docker run	Crea e inicia un contenedor a partir de una imagen.	Despliegue
nmap	Herramienta de escaneo de redes para descubrir puertos y servicios.	Auditoría
hydra	Herramienta para realizar ataques de fuerza bruta a servicios de red.	Ataque

# 🧠 Conclusiones y Aprendizajes
Esta práctica permitió aplicar herramientas fundamentales en ejercicios de hacking ético, destacando la importancia de dominar conceptos como la construcción de imágenes Docker, la administración de contenedores, el escaneo de servicios y la explotación mediante ataques de diccionario.

Entre los aprendizajes obtenidos se destaca:

- La capacidad de personalizar un entorno vulnerable para pruebas controladas.

- La identificación y validación de servicios activos a través de Nmap.

- El uso de Hydra como herramienta efectiva para comprometer credenciales débiles en servicios como FTP.


