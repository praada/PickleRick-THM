# Máquina PICKLE RICK de TryHackMe en Español
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/7d3e7597-e3c2-49da-a53d-1da99a9f7b46" alt="portada">
</p>

Estoy subiendo mi primer Write Up resolviendo una máquina CTF de la plataforma TryHackMe recomendada para principiantes que se interesan por aprender sobre hacking ético/Pentesting.

Tenemos esta máquina como dije anteriormente de dificultad **fácil**, en la máquina como principiantes vemos tanto la parte de Reconocimiento, Enumeración, Recuperar los tres ingredientes (Explotación de vulnerabilidades), Escalada de Privilegio (por decir) y Reverse Shell.
Utilice estas Herramientas para realizar la maquina:
- nmap
- Whatweb
- Gobuster
- Burpsuite

Una vez que la máquina esté iniciada verificamos con un ping para verificar si dicha máquina está encendida/activa.
```css
ping -c 1 10.10.46.34
PING 10.10.46.34 (10.10.46.34) 56(84) bytes of data.
64 bytes from 10.10.46.34: icmp_seq=1 ttl=61 time=142 ms

--- 10.10.46.34 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 141.940/141.940/141.940/0.000 ms
```
Gracias al TTL=61 tenemos una idea de que la máquina sea **LINUX**  

La máquina esta activa, se procede a hacer el Reconocimientos de Puertos con la herramienta NMAP.

```css
nmap -p- --open -sS --min-rate 2000 -vvv -n -Pn 10.10.46.34 -oG thePorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-07 09:33 EST
Initiating SYN Stealth Scan at 09:33
Scanning 10.10.46.34 [65535 ports]
Discovered open port 22/tcp on 10.10.46.34
Discovered open port 80/tcp on 10.10.46.34
Completed SYN Stealth Scan at 09:34, 59.06s elapsed (65535 total ports)
Nmap scan report for 10.10.46.34
Host is up, received user-set (0.15s latency).
Scanned at 2024-05-07 09:33:40 EST for 59s
Not shown: 50732 filtered tcp ports (no-response), 14801 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 61
80/tcp open  http    syn-ack ttl 61

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 59.14 seconds
           Raw packets sent: 117612 (5.175MB) | Rcvd: 15699 (628.032KB)
```
Finalizado el escaneo nos informa que se encuentran dos puertos abiertos
- 22: Protocolo SSH que utiliza para el acceso remoto seguro a sistemas y transferencia de archivos.
- 80: Protocolo HTTP utilizado para servir páginas web.  

Vamos a Reconocer/Enumerar las versiones de los puertos abiertos encontrados
```css
nmap -sC -sV -p22,80 10.10.46.34 -oN target                             
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-07 09:38 EST
Nmap scan report for 10.10.46.34
Host is up (0.14s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:00:4b:82:6e:9c:91:fe:dc:d3:86:27:cc:7f:ab:23 (RSA)
|   256 e3:73:17:4e:fe:2b:c7:60:00:82:52:83:da:be:6b:69 (ECDSA)
|_  256 a7:24:f6:f4:b7:b7:9e:5d:c1:48:76:43:c9:bd:a6:20 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.37 seconds
```

Donde tenemos resumido las versiones de los puertos abiertos
```css
22-->OpenSSH 8.2p1 Ubuntu
80-->Apache httpd 2.4.41
```
  
Con Whatweb vamos a ver las tegnologias que trabajan en el puerto 80
```python
http://10.10.46.34 [200 OK] Apache[2.4.41], Bootstrap, Country[RESERVED][ZZ], HTML5,
HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.46.34], JQuery, Script, Title[Rick is sup4r cool]
```

Asi que vamos a entrar a dicho puerto en nuestro navegador para investigar la pagina web
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/9a0696c4-39e2-4847-91d4-e35853a82b5b" alt="source">
</p>

Una buena practica es leer el source-code de la pagina, ya que aveces hay brechas de seguridad en comentarios no eliminados
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/d920c81c-6f0f-427d-9317-d23014fe0fd1" alt="source">
</p>

Como en este caso de la maquina que en un comentario estaba almacenado el usuario. Ya tenemos una credencial de usuario y faltaria una password, pero para que? de seguro hay un panel de acceso o algo asi, no creo que sea por SSH. Tambien no se puede dejar pasar que vemos que hay un directorio "assets/".

Continuamos con la enumeracion de directorios con **Gobuster** y tambien archivos con algunas extensiones definidas.
```python
gobuster dir -u http://10.10.46.34 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 5 -x txt,js,php,py 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.46.34
[+] Method:                  GET
[+] Threads:                 5
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,js,php,py
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 276]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 311] [--> http://10.10.46.34/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
Progress: 14274 / 1102805 (1.29%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 14287 / 1102805 (1.30%)
===============================================================
Finished
===============================================================
```

Enumerados los directorios/archivos con gobuster empezamos a investigar.

/assets
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/3ebe5726-72e9-4b1a-bb41-6bc412e5fbf0" alt="gobuster">
</p>

No encontro nada interesante, tampoco en el source-code, seguimos...

/robots.txt
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/d0919751-0816-4a6b-83b2-7506f6676fb1" alt="robots">
</p>

En el archivo .txt encontramos algo que puede ser una credencial ya sea un Usuario o Password. Esto ya nos da un indicio si se usa para conectarse por SSH o en el login.php enumerado con gobuster

/login.php
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/90a87088-5e58-4812-a56f-0c6e64901527)" alt="login">
</p>

Encontramos un portal de autenticacion, teniendo 2 credenciales Usuario: **R1ckRul3s** y **Wubbalubbadubdub** que lo usare como si fuera una password.

Y se accedio al sistema que es como un panel de Comandos, donde tiramos un "whoami" y nos dice que somos el usuario **www-data**
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/df1445dd-23a5-4bb9-bd9b-de178e901751" alt="panel">
</p>

Se procede a listar en la ruta actual ($pwd ls -l) todos los archivos con sus permisos
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/3fa4b78b-04db-4393-91a8-76f5d813dc2a)" alt="($pwd) ls -l">
</p>

Se encontro el archivo que contiene la flag del CTF que es el primer ingrediente en un archivo .txt
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/037d675e-ba7c-4bcc-b82d-a76b316268a7)" alt="cat 1flag">
</p>
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/430a2ea7-0780-478f-a6fa-09645ea7dc99)" alt="error">
</p>

Pero cat al parecer esta filtrado, pero algunas variaciones a cat como tac, more, less, head, etc. tal vez funcionen pero usare Burpsuite para capturar la peticion.

Mi idea seria bypassear ese filtro, se intento URL Encodear **cat** pero no funciono
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/98bdc2cc-0353-4908-a5bd-649a50a2558e" alt="burp1">
</p>

Despues de varios intentos se bypasseo utilizando caracteres de escape como seria en este caso (" \\ ") la barra invertida que evita la seguridad/filtrado de la maquina
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/2e651e7c-7d8b-47ec-967c-b9c5260ea958" alt="burp2">
</p>
Encontramos la Primera flag del CTF que es el primer ingrediente

Continuando se verifica el otro archivo .txt para ver si tiene el segundo ingrediente
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/55dc3802-fec9-4687-a693-6bbe7888a3c5" alt="burp3">
</p>
Pero solo dio una pista, como debemos buscar a traves del sistema vamos a establecer una Reverse Shell para trabajar de mejor manera. Esto se logra en mi caso verificando que tenia instalado php.

Esto se URL Encodeo, para evitar problemas con Burpsuite. Recomendo esta web de [PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/9b955252-c2e3-4456-874e-76e1c999d769" alt="burp4">
</p>
Se envia la peticion maliciosa y tenemos la Reverse Shell
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/acd1738c-b71a-4aba-8cf6-314cd3f5c23c" alt="burp4">
</p>

Una vez de settear la terminal y buscar por el sistema
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/dfa6e81d-3a7e-40b0-b7c6-160f66d45fae" alt="rce1">
</p>
en el directorio /home/rick tenemos la segunda flag de este **CTF**, aun falta la ultima flag

La busqueda permisos SUID en archivos no fue exitosa
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/e0b9bb55-2147-4270-b7a6-b7a608365fd1" alt="rce2">
</p>

Se procede a listar los permisos del usuario
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/95eedbae-f617-4e48-83a2-531bded31799" alt="rce3">
</p>
Pues al parecer el usuario root no tiene password. Y la escalada de privilegios fue exitosa
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/9c1ec65f-b968-40d0-93c7-84311a23fdec" alt="rce4">
</p>

Ahora seria buscar en el directorio root la ultima flag del CTF
<p align="center">
  <img src="https://github.com/praada/PickleRick-THM/assets/82685678/50a839fa-6837-49b1-b7db-bb73bd445c8f" alt="rce5">
</p>
Tenemos la tercera flag de este **CTF**

FIN!
