# Máquina WhoAmI

---------------------

Dificultad -> Medium

Enlace a la máquina -> [Dockerlabs](https://dockerlabs.es/)

## 1. Nmap

Primero, realizamos un escaneo con nmap para ver puertos y servicios abiertos

```shell
nmap -p- --open -sV -sC -sS --min-rate=5000 -n -Pn 172.18.0.2
------------------------------------------------------------
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Whoiam
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-generator: WordPress 6.5.4
MAC Address: 02:42:AC:12:00:02 (Unknown)
```

Vemos que hay un Apache el puerto 80 asi que nos metemos a la página web

![](.assets/1cb63fb5d2eedd3e93b3946bb3a59b06d156795a.png)
