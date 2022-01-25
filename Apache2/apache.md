# Hárdening de un servidor Apache

Para empezar con la instalación y configuración de un servidor Apache, primero aseguraremos que el gestor de paquetes actualice los repositorios de paquetes.

Después instalaremos los paquetes de Apache.
```bash
# apt update
# apt install apache2
```
Podemos ver si el servicio está funcionando correctamente después de instalarlo ejecutando
```bash
# service apache2 status
```
El output debería ser similar a este:
![Estado del servicio](Apache2/Imagenes/itworks!.png)

Como se puede ver, es conveniente quitar el index.html original y modificarlo con un documento de prueba. El index.html por defecto, tanto de PHP como de Apache, conllevan información sensible que no puede ser vista por el público.

Todos estos documentos por defecto se encuentran en 
```\var\www\html ```, así que podemos sustituir su contenido con

```bash
# echo "Hello World!" > /var/www/html/index.html
```
