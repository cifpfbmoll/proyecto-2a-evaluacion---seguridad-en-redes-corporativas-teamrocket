# Apache2

## Introducción

Apache es un servidor web ampliamante conocido por su modularidad y respaldado por una organización enfocada en el software open-source nacido en 1995. Su competidor directo es Nginx el cual trabaja de forma más eficiente a la hora de gestionar grandes cantidades de solicitudes ya que es multi-hilo.

## Instalación

Para proceder a la instalación de apache2 simplemente:

```bash
$sudo apt update
$sudo apt install apache2
```

## Configuración

Toda la configuración es realizada desde la ruta /etc/apache2. Dentro encontraremos principalmente:

+ Ficheros
    + apache2.conf -> Configuración general de apache
    + ennvars -> Variables de entorno
    + ports.conf -> Llamado desde apache2.conf donde se definen los puertos de escucha por defecto
+ Directorios
    + sites-enabled/available -> Ubicación de los sitios (vhosts)
    + conf-enabled/available -> Ubicación de las configuraciones
    + mods-enabled/available -> Ubicación de los módulos

![ficheros](/img/1.png)

### Por defecto

Normalmente la ruta por defecto de la web de apache2 es `/var/www/html`, normalmente trae un index.html con información acerca de apache2. El cual hemos editado y se ve de la siguiente forma.

![imagen1](/img/index-html.png)

### Usuario y grupo específico

Por norma general apache define un usuario y grupo `www-data` específicos para que en caso de una filtración de credenciales de dicho usuario, este quede sujeto unicamente a aquello a lo que se le han otorgado permisos. Es decir, solo tendrá acceso a las webs y configuraciones del propio Apache.

Este usuario es configurable desde el archivo `ennvars`

```
export apache_run_user= www-data
export apache_run_group= www-data
```

### Ocultar información del servidor en el header

Debido a que la primera fase de cualquier ataque se basa a grandes rasgos en el reconocimiento. Es necesario hacer lo más díficil posible que el atacante sea capaz de sacar información sobre la que pueda basar su ataque, como pueden ser la versiones del software.

Desde la ruta `/etc/apache2/conf-enabled/security.conf` podemos editar dos líneas de forma que ya no sea visible la versión del servidor web. Al menos no de forma directa desde un navegador convencional.

```
ServerTokens Prod
ServerSignature Off
```

![imagen2](/img/header-update.png)

### Desactivar módulos no usados

Como el principio de seguridad se basa en gran parte en reducir la superficie de ataque, deberíamos desactivar los módulos que no esten siendo utilizados por ninguna web. Esto se puede realizar utilizando el siguiente comando.

```bash
#Mostrar modulos
$apache2ctl -M

#Desactivar modulo
$a2dismod nombre_modulo

#Reiniciar apache2
$service apache2 restart
```

### Virtualhost

**Que es?**

Imágina que por cada web que hay en internet, hubiese un servidor dedicado a ella con una única web hospedada. No tiene sentido, verdad? Pues para evitar que eso fuese así apache aplica una serie de reglas en la configuración de los sitios que al detectar un "ServerName" específico, en el Header de la solicitud del cliente, te dirigen a una de las webs que hospeda o a otra basándose en el dominio.

Para generar un virtualhost lo único que debemos hacer es crear un archivo en "/etc/apache2/sites-enabled" tal que:

```xml
<VirtualHost *:80> 

	ServerName server.yuki.ej
	ServerAlias www.dominio2.ej cococabra.leche.gen
	
	DocumentRoot /var/www/web_ejemplo	

	<Directory /var/www/web_ejemplo>
		Options -Indexes FollowSymLinks MultiViews
		Require all granted
    Require not ip 10.252.46.165		
	</Directory>

	Redirect /no  https://yukics.wordpress.com

	ErrorDocument 403 /e403.html #El error 403 devolvera el siguiente archivo html
	ErrorDocument 404 "hahahahaha 404" # El error 404 mostrara ese texto
	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

| Directiva | Descripción |
| --- | --- |
| ServerName | Nombre con el que el servidor se identifica a si mismo |
| ServerAlias | Los nombres, dominios, que contrasta el vhost a la hora de contestar una solicitud |
| DocumentRoot | Ubicación de los archivos que componen la web del sitio |
| Directory | Se utiliza para especificar configuraciones específicas dentro de los directorios especificados |
| Options  | Define una serie de parametros relativos al comportamiento del sitio, para aplicar el contrario definir un "-*" |
| Require | Define el acceso al recurso, en este caso a la web en sí, puede hacerse, por "ip|host" o de forma genérica. Se aplica en orden descendente. |
| Redirect | Se utiliza para redirigirte a otra web |
| ErrorDocument | Acompañado del código de estado define el texto o archivo con el que contesta una solicitud en caso de que ocurra ese código de estado. |
| ErrorLog | Ubicación del log únicamente de errores |
| CustomLog | Ubicación del logs relacionado al funcionamiento de apache |

**Autenticación básica**

Existen dos formas de realizar una autenticación, no excesivamente segura cabe destacar. Pero que para salir del paso funciona. Utilizando dos métodos: digest y basic. Funcionan de forma similar y la única diferencia es el comando utilizado para generar el fichero de autenticación y que el digest implementa grupos.

Hoy en día se aconseja la Basic frente a la Digest ya que implementa un cifrado más robusto del pass. Antiguamente era al reves ya que basic guardaba el pass en texto plano.

**BasicAuth**

```bash
#Crear ficherox y definir usuario
$htpasswd -c ficherox user1

#Agregar usuario a ficherox
$htpasswd ficherox user2
```

```xml
...
<Directory /var/www/web_ejemplo/restringido>
    Options Indexes FollowSymLinks MultiViews
    AuthUserFile "/etc/apache2/ficherox" 
    AuthName "Login super seguro"
    AuthType Basic 
    Require valid-user
    Order deny,allow
    Allow from 127.0.0.1, 192.168.1.0/24
</Directory>
...
```

| Directiva | Descripción |
| --- | --- |
| AuthUserFile  | Ruta absoluta del archivo generado previamente "ficherox" |
| AuthName | Nombre cualquiera que aparecerá en la ventana de login |
| AuthType  | Tipo de acceso en este caso basic |
| Require valid-user | Obliga a que el usuario sea válido para acceder |
| Order deny,allow | Deniega el acceso por defecto y habilita los que especifiquemos |
| Allow from 127.0.0.1, 192.168.1.0/24 | Permite el login desde localhost y desde la red de mi casa, por ejemplo. |

**DigestAuth**

```bash
#Crear ficherox2 y definir usuario
htdigest -c ficherox2 grupox admin1

#Agregar usuario a ficherox2
htdigest ficherox2 grupox admin2
```

```xml
<Directory /var/www/profesor/departamento>
    Options Indexes FollowSymLinks MultiViews
    AuthUserFile "/etc/apache2/ficherox2"
    AuthName "grupox"
    AuthType Digest
    Require valid-user
    Order deny,allow
    Allow from 127.0.0.1, 172.16.242.1
</Directory>
```

Los únicos cambios respecto a Basic son:

| Directiva | Descripción |
| --- | --- |
| AuthName | En digest debe coincidir con el nombre del grupo al que se asigna el usuario |
| AuthType  | Tipo de acceso en este caso digest |

**Certificados SSL**

```xml
...

SSLCertificateFile "/etc/apache2/ssl/algo.crt"
SSLCertificateKeyFile "/etc/apache2/ssl/algo.key"
```