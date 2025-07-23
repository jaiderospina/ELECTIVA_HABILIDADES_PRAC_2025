En nuestra anterior práctica hicimos uso de un contenedor para crear un recurso que fuera explotable mediante **ataque de diccionario**  al servicio SSH. 

Acá se presenta el procedimiento previo realizado para crear la imágen  con el servicio mencionado, y compartirlo mediante Dockerhub.

# Actividad Final. 

En los grupos de trabajo y guíandose por este ejemplo construir una imágen vulnerable de un servicio FTP ( el contenedor deberá tener instalado este servcio), subir la imágen a Docker  y recrear en el Overview  de Docker hub:

- Próposito de la imágen.
- Reto (autenticarse al servicio).
- Resolución del reto.
- En clase se debe presentar el reto y ejecutarlo desde:
   * Descargue de la imágen.
   * Ejecución del contenedor.
   * Creación del diccionario.
   * Realización del ataque.
   * Consumo del servicio (ataque-autenticación).
   * **Nota**  Dado que todo esta documentado en el Overview de github, solo es cuestión de "copiar" y "pegar".
      

  **NOTA**:  Como se trata de emular un CTF y como el servicio se rompera mediante ataque de diccionario, se deben presentar las pistas que permitan que el participante pueda construir de manera eficiente el diccionario.



---

**Ejemplo recurso CTF**

### 1\. Dockerfile para Ubuntu con SSH y Usuario Personalizado

Para iniciar, se requiere un archivo llamado `Dockerfile`. Este archivo se crea en una carpeta vacía y se le añade el siguiente contenido. 

```dockerfile
# Se utiliza una imagen base de Ubuntu
FROM ubuntu:latest

# Se instala OpenSSH-server y otras dependencias necesarias
RUN apt-get update && \
    apt-get install -y openssh-server sudo && \
    mkdir /var/run/sshd

# Se configura SSH para permitir el inicio de sesión root (útil para depuración inicial, aunque se recomienda deshabilitar en producción por seguridad)
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/UsePAM yes/UsePAM no/' /etc/ssh/sshd_config

# Se crea el usuario 'Legion' con la contraseña 'Esdeg' y se le añade al grupo sudo
RUN useradd -rm -d /home/legion -s /bin/bash -g root -G sudo legion && \
    echo "legion:Esdeg" | chpasswd

# Se expone el puerto SSH
EXPOSE 22

# Este comando inicia el servicio SSH cuando el contenedor se ejecuta
CMD ["/usr/sbin/sshd", "-D"]
```

-----

### 2\. Procedimiento para Crear el Contenedor (Construcción de la Imagen)

Para llevar a cabo este proceso, se debe asegurar que Docker Desktop esté instalado y en ejecución en el sistema Windows.

1.  **Abrir una terminal:** Se recomienda usar PowerShell o Símbolo del sistema.
2.  **Navegar al directorio del Dockerfile:** Utilizar el comando `cd` para ir a la carpeta donde se guardó el `Dockerfile`.
    ```bash
    cd C:\ruta\a\la\carpeta_docker
    ```
3.  **Construir la imagen:** Ejecutar el comando `docker build`. Se le asignará el nombre `retoctf` y la etiqueta `1.0`. El `.` al final indica que el Dockerfile se encuentra en el directorio actual.
    ```bash
    docker build -t retoctf:1.0 .
    ```
    Este proceso descargará la imagen base de Ubuntu, instalará SSH, creará el usuario y configurará todo según el Dockerfile. La primera vez, puede tardar unos minutos.
4.  **Verificar la creación de la imagen:**
    ```bash
    docker images
    ```
    Se debería ver `retoctf` listada entre las imágenes.

-----

### 3\. Procedimiento para Publicar la Imagen en Docker Hub

Para publicar la imagen, se necesita una cuenta en Docker Hub. Si no se tiene una, se puede crear en [hub.docker.com](https://hub.docker.com/).

1.  **Iniciar sesión en Docker Hub desde la terminal:**
    ```bash
    docker login
    ```
    Se solicitará el nombre de usuario y la contraseña de Docker Hub.
2.  **Etiquetar la imagen para Docker Hub:** La imagen necesita tener un nombre que incluya el nombre de usuario de Docker Hub. Se debe reemplazar `tu_usuario_dockerhub` con el nombre de usuario real.
    ```bash
    docker tag mi-ssh-ubuntu:1.0 tu_usuario_dockerhub/mi-ssh-ubuntu:1.0
    ```
3.  **Enviar (push) la imagen a Docker Hub:**
    ```bash
    docker push tu_usuario_dockerhub/mi-ssh-ubuntu:1.0
    ```
    Este comando subirá la imagen al repositorio público en Docker Hub. Una vez completado, será visible en el perfil de Docker Hub.

-----

### 4\. Procedimiento para Descargar la Imagen y Ejecutar el Contenedor

Si se desea utilizar esta imagen en otra máquina o simular la descarga, se seguirán los siguientes pasos:

1.  **Asegurarse de que la imagen local no exista (opcional):** Para simular una descarga, primero se puede eliminar la imagen localmente.
    ```bash
    docker rmi tu_usuario_dockerhub/mi-ssh-ubuntu:1.0
    docker rmi mi-ssh-ubuntu:1.0
    ```
2.  **Descargar la imagen desde Docker Hub:**
    ```bash
    docker pull tu_usuario_dockerhub/mi-ssh-ubuntu:1.0
    ```
    Esto descargará la imagen previamente publicada.
3.  **Ejecutar el contenedor en segundo plano (modo detached):**
    Se utiliza la opción `-d` para ejecutar el contenedor en segundo plano y `-p` para mapear el puerto 22 del contenedor a un puerto disponible en la máquina local (por ejemplo, 2222).
    ```bash
    docker run -d -p 2222:22 --name mi-contenedor-ssh tu_usuario_dockerhub/mi-ssh-ubuntu:1.0
    ```
      * `-d`: Ejecuta el contenedor en modo *detached* (segundo plano).
      * `-p 2222:22`: Mapea el puerto 22 del contenedor al puerto 2222 de la máquina local. Se puede elegir cualquier puerto libre en la máquina local en lugar de 2222.
      * `--name mi-contenedor-ssh`: Asigna un nombre fácil de recordar al contenedor.
      * `tu_usuario_dockerhub/mi-ssh-ubuntu:1.0`: La imagen que se acaba de descargar.
4.  **Verificar que el contenedor esté corriendo:**
    ```bash
    docker ps
    ```
    Se debería ver el contenedor `mi-contenedor-ssh` listado con el puerto 2222/tcp mapeado.

-----

### 5\. Conectarse al Contenedor Vía SSH

Una vez que el contenedor está en ejecución, se puede conectar a él utilizando cualquier cliente SSH (como PuTTY en Windows o el cliente SSH integrado en PowerShell/Símbolo del sistema).

**Desde PowerShell o Símbolo del sistema (Windows 10/11):**

```bash
ssh legion@localhost -p 2222
```

  * `legion`: El nombre de usuario creado en el Dockerfile.
  * `localhost`: La dirección IP de la máquina local, ya que se está conectando al puerto mapeado de Docker.
  * `-p 2222`: El puerto al que se mapeó el puerto 22 del contenedor.

Cuando se solicite la contraseña, se debe introducir **Esdeg**.

**Primera conexión:** La primera vez que se conecte, es posible que se reciba una advertencia sobre la autenticidad del host. Se debe escribir `yes` para continuar.

¡Listo\! Ahora se está conectado por SSH al contenedor de Ubuntu. Para salir de la sesión SSH, se utiliza el comando `exit`.

Para detener el contenedor:

```bash
docker stop mi-contenedor-ssh
```

Para eliminar el contenedor (después de detenerlo):

```bash
docker rm mi-contenedor-ssh
```

