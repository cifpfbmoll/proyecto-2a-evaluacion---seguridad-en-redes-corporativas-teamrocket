# Apache2

## Introducción

## Instalación

Para proceder a la instalación de apache2 simplemente:

```bash
$sudo apt update
$sudo apt install apache2
```

## Configuración

Toda la configuración es realizada desde la ruta /etc/apache2, al menos en Linux. Dentro encontraremos principalmente:

+ Ficheros
    + apache2.conf -> Configuración general de apache
    + ennvars -> Variables de entorno
+ Directorios
    + sites-enabled/available -> Ubicación de los sitios  (vhosts)
    + conf-enabled/available -> Ubicación de las configuraciones
    + mods-enabled/available -> Ubicación de los módulos

### Por defecto

Normalmente la ruta por defecto de la web de apache2 es `/var/www/html`, normalmente trae un index.html con información acerca de apache2. El cual hemos editado y se ve de la siguiente forma.

![imagen](/img/index-html.png)

### Usuario y grupo específico

Por norma general apache define un usuario y grupo `www-data` específicos para que en caso de una filtración de credenciales de dicho usuario, este quede sujeto unicamente a aquello a lo que se le han otorgado permisos. Es decir, solo tendrá acceso a las webs y configuraciones del propio Apache.

Este usuario es configurable desde el archivo `ennvars`

```
export apache_run_user= www-data
export apache_run_group= www-data
```