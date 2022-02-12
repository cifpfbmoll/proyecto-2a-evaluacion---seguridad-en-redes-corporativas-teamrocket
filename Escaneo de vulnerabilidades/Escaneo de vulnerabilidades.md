# Reconociemiento

Como ya aseguramos nuestro servidor en las anteriores actividades de este proyecto ni nessus ni openVas detectan vulnerabilidades, simplemente nos informan de que nos certificados utilzados en nuestro servicio web son autofirmados. Así que procederemos a la deteccción y explotación de vulnerabilidades directamente en Metasploitable2. A continuación imágenes del escaneo a este contenedor, para ello simplemente hicimos un `docker run` definiendo la red de docker en la que queriamos que se crease con un `--network`.

![nessus](img/1.png)
![openvas](img/2.png)