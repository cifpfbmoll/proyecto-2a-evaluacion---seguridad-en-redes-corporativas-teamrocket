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

