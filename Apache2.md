# Apache2

### Indice
1. [Introducción](#introducción)
2. [Instalación](#instalación)
3. [Configuración](#conf)
    1. [Por defecto](#default)
    2. [Usuario y grupo](#user)
    3. [Ocultar información del servidor en el header](#noinfo)
    4. [Desactivar módulos no usados](#dismod)
    5. [VirtualHosts](#vhost)
        1. [Autenticación](#auth)
        1. [Certificados](#certs)
    5. [.htacces](#htacces)
    6. [Hotlinking](#hot)
4. [ModSecurity](#mod)
    1. [Instalación](#modinstall)
    2. [Configuración](#modconf)
    3. [Reglas OWASP](#owasp)
        1. [Teoría OWASP](#towasp)
    4. [Pentesting](#testmod)
    

## Introducción<a name="introducción"></a>

Apache es un servidor web ampliamante conocido por su modularidad y respaldado por una organización enfocada en el software open-source nacido en 1995. Su competidor directo es Nginx el cual trabaja de forma más eficiente a la hora de gestionar grandes cantidades de solicitudes ya que es multi-hilo.

## Instalación<a name="instalación"></a>

Para proceder a la instalación de apache2 simplemente:

```bash
$sudo apt update
$sudo apt install apache2
```

## Configuración<a name="conf"></a>

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

### Por defecto<a name="default"></a>

Normalmente la ruta por defecto de la web de apache2 es `/var/www/html`, normalmente trae un index.html con información acerca de apache2. El cual hemos editado y se ve de la siguiente forma.

![imagen1](/img/2.png)

### Usuario y grupo específico<a name="user"></a>

Por norma general apache define un usuario y grupo `www-data` específicos para que en caso de una filtración de credenciales de dicho usuario, este quede sujeto unicamente a aquello a lo que se le han otorgado permisos. Es decir, solo tendrá acceso a las webs y configuraciones del propio Apache. Por lo que cualquier contenido que deba ser visible por apache2 deberemos aplicarlo con un `sudo chown www-data:www-data /var/www/[new directory] && sudo chmod -R 775 /var/www/[new directory]` 

Este usuario es configurable desde el archivo `ennvars` editando las siguientes líneas:

```
export apache_run_user= www-data
export apache_run_group= www-data
```

### Ocultar información del servidor en el header<a name="noinfo"></a>

Debido a que [la primera fase de cualquier ataque](https://www.incibe.es/protege-tu-empresa/blog/las-7-fases-ciberataque-las-conoces) se basa a grandes rasgos en el reconocimiento. Es necesario hacer lo más díficil posible que el atacante sea capaz de sacar información sobre la que pueda basar su ataque, como pueden ser la versiones del software.

Desde la ruta `/etc/apache2/conf-enabled/security.conf` podemos editar dos líneas de forma que ya no sea visible la versión del servidor web. Al menos no de forma directa desde un navegador convencional.

```
ServerTokens Prod
ServerSignature Off
```

![imagen2](/img/3.png)

### Desactivar módulos no usados<a name="dismod"></a>

Como el principio de seguridad se basa en gran parte en reducir la superficie de ataque, deberíamos desactivar los módulos que no esten siendo utilizados por ninguna web. Esto se puede realizar utilizando el siguiente comando.

```bash
#Mostrar modulos
$apache2ctl -M

#Desactivar modulo
$a2dismod nombre_modulo

#Reiniciar apache2
$service apache2 restart
```

### Virtualhost<a name="vhost"></a>

**Que es?**

Imágina que por cada web que hay en internet, hubiese un servidor dedicado a ella con una única web hospedada. No tiene sentido, verdad? Pues para evitar que eso fuese así apache aplica una serie de reglas en la configuración de los sitios que al detectar un "ServerName" específico, en el Header de la solicitud del cliente, te dirigen a una de las webs que hospeda u otra basándose en ese nombre (por norma general un dominio).

Para generar un virtualhost lo único que debemos hacer es crear un archivo en "/etc/apache2/sites-available" con la sintaxis descrita a continuación, luego aplicar la configuración con el comando `a2ensite nombre_del_vhost && service apache2 reload`:

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
| [Options](https://docstore.mik.ua/orelly/linux/apache/ch03_11.htm)  | Define una serie de parametros relativos al comportamiento del sitio, para aplicar el contrario definir un "-*" |
| Require | Define el acceso al recurso, en este caso a la web en sí, puede hacerse, por "ip|host" o de forma genérica. Se aplica en orden descendente. |
| Redirect | Se utiliza para redirigirte a otra web |
| ErrorDocument | Acompañado del código de estado define el texto o archivo con el que contesta una solicitud en caso de que ocurra ese código de estado. |
| ErrorLog | Ubicación del log únicamente de errores |
| CustomLog | Ubicación del logs relacionado al funcionamiento de apache |

**Autenticación básica**<a name="auth"></a>

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

**Certificados SSL**<a name="certs"></a>

Podemos generar certificados con [certbot](https://certbot.eff.org/instructions?ws=apache&os=ubuntufocal) o bien con el comando `openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout algo.key -out algo.crt`, y moverlos a cualquier ruta "segura", en este ejemplo es `/etc/apache2/ssl`. Por último activar el modulo ssl y aplicar SSL desde el vhost.

```bash
$a2enmod ssl
```

```xml
...

SSLCertificateFile "/etc/apache2/ssl/algo.crt"
SSLCertificateKeyFile "/etc/apache2/ssl/algo.key"
```

Una vez configurado ssl en un vhost por el puerto 443, podemos aplicar una redireccion desde el vhost puerto 80, para obligar a que las conexiones sean siempre encriptadas, la forma más limpias es con un:

```xml
<VirtualHost *:80> 

	ServerName server.yuki.ej
	Redirect permanent / https://server.yuki.ej
	...
</VirtualHost>

<VirtualHost *:443> 

	ServerName server.yuki.ej
	...
</VirtualHost>
```

![ssl](/img/4.png)

### .htacces<a name="htacces"></a>

El archivo htaccess (acceso de hipertexto) es un archivo oculto, ubicado en el directorio de cada site, que se utiliza para configurar funciones adicionales para sitios web alojados en el servidor web Apache. Con él, puedes reescribir la URL, proteger directorios con contraseña, habilitar la protección de enlaces directos, no permitir el acceso a direcciones IP específicas, cambiar la zona horaria de tu sitio web o alterar la página de índice predeterminada, y mucho más. Realmente puedes configurar casi lo mismo que con un VirtualHost pero sin tener que editar la configuración de apache2 directamente, esto posibilita otorgar acceso a un cliente y que él mismo defina la configuración de su sitio, por poner un ejemplo de su utilidad.

### Hotlinking<a name="hot"></a>

El hotlinking es una forma de mostrar recursos de una web sin necesariamente poseer ese recurso, esto seria un ejemplo: `<img src="https//:google.com/logo.png">`. Lo que ocurre con esta práctica es que consume una serie de recursos y no siempre nos interesará permitirlo. Una forma muy práctica de evitarlo es:

1. Activar el modulo rewrite

    ```bash
    $a2enmod rewrite && service apache2 restart
    ```

2. Editar o el .htacces o el vhost y agregar los sitios que no queremos bloquear y por ultimo bloquear el hotlinking a todas las imagenes de nuestro sitio:

    ```
    RewriteCond %{HTTP_REFERER} !^$
    RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?yourwebsite.com [NC]
    RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?google.com [NC]
    RewriteCond %{HTTP_REFERER} !^http(s)?://(www\.)?facebook.com [NC]
    RewriteRule \.(jpg|jpeg|png|gif)$ - [F]
    ```
## mod_security<a name="mod"></a>

ModSecurity es un WAF (Web Application Firewall), con el seremos capaces de detectar ataques diversos basandose en reglas que definamos.

### Instalación<a name="modinstall"></a>

```bash
$apt install libapache2-mod-security2
$a2enmod security2
$systemctl restart apache2
```

### Configuración<a name="modconf"></a>

Renombrar archivo a *.conf, de esta forma el modulo será capaz de leer la configuración anteriormente desactivada. Reiniciamos apache2 después y en principio debería funcionar, aunque de forma rudimentaria.

```bash
$mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
$sudo service apache2 restart
```
*Cargar el módulo en apache2.conf si no encontramos el archivo security2.load en la carpeta mods-enabled, que se encarga de cargar el módulo:*

```bash
#Agregar 
LoadModule security2_module modules/mod_security2.so
```

Se supone que debemos acabar de configurar el waf editando la configuración que antes hemos renombrado en `/etc/modsecurity`. [Referencia](https://www.linuxbabe.com/security/modsecurity-apache-debian-ubuntu)

```bash
$sudo nano /etc/modsecurity/modsecurity.conf

#Definir las siguientes directivas
SecRuleEngine On #Bloquear ataques http
SecAuditLogParts ABCEFHJKZ #Editar configuración de logs
```

### Instalar reglas OWASP(CRS)<a name="owasp"></a>

Descargar las reglas de Github y extraerlas:

```bash
#Las mas actualizadas cuando se hizo la guía eran las 3.3.2, revisar posibles actualizaciones
$wget https://github.com/coreruleset/coreruleset/archive/v3.3.2.tar.gz
$tar xvf v3.3.2.tar.gz
```

Creamos un directorio para las reglas y las movemos ahí, también las podemos poner en otros lugares como `/etc/modsecurity/crs`:

```bash
$sudo mkdir /etc/apache2/modsecurity-crs/
$sudo mv coreruleset-3.3.2 /etc/apache2/modsevurity-crs
```

A continuación editar el archivo de configuración que se encuentra desactivado e incluimos en la configuración de modsecurity al paquete de reglas:

```bash
$cd /etc/apache2/modsecurity-crs/coreruleset-3.3.2/
$sudo cp crs-setup.conf.example crs-setup.conf
$sudo nano /etc/apache2/mods-enabled/security2.conf

#Agregamos esto al documento de igual forma que lo hemos hecho antes
IncludeOptional /etc/apache2/modsecurity-crs/coreruleset-3.3.0/*.conf
Include /etc/apache2/modsecurity-crs/coreruleset-3.3.0/rules/*.conf
```

Comprobamos las configuraciones y reinciamos el servicio

```bash
$sudo apache2ctl -t
$sudo service apache2 restart 
```

**Teoría**<a name="towasp"></a>

Desde el archivo de configuración que acabamos de editar de `crs-setup.conf` podemos definir un nivel de paranoia, esto se traduce en lo agresivo que se comporte el WAF. O simplemente activar o desactivar reglas. El archivo está bien dodcumentado. Y también decir que el modulo puede trabajar de dos formas `self-contained mode` o `anomaly scoring mode`, este último es el que viene por defecto con esta versión.

Ahora habilitaremos la regla de prevención de DOS, también podemos modificarla:

```bash
$sudo nano /etc/apache2/modsecurity-crs/coreruleset-3.3.2/crs-setup.conf
```

![enable dos protect](/img/8.png)

Los logs serán depositados en el siguiente archivo `/var/log/apache2/modsec_audit.log`. En el siguiente ejemplo vemos como es capaz de detectar y bloquear un intento de ejecución remota de códgio bastante simple `http://127.0.0.1/index.html?exec=/bin/bash`:

![remote shell](/img/7.png)

### Pruebas a mod_security<a name="testmod"></a>

**[Slowloris](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/auxiliary/dos/http/slowloris.md)**

Es una herramienta del framework metasploit utilizada para hacer ataques DOS hacia:

+ Apache HTTP Server 1.x and 2.x
+ Apache Tomcat 5.5.0 through 5.5.29, 6.0.0 through 6.0.27 and 7.0.0 beta

```bash
$sudo msfconsole
>use auxiliary/dos/http/slowloris
>show options #para ver la configuraciond el modulo
>set rhost 10.0.0.10
>set rport 443
>set ssl true
>set sockets 1000 #aunque la mv solo podrá tirar 225 por sus limitaciones de hw
>run #Ejecuta el ataque
```

En las siguientes imágenes se puede apreciar como nada más empezar el ataque el servidor apache2 deja de responder la solicitud de mi navegador, en cuando se detiene el ataque el navegador vuelve a ser capaz de acceder a la web.

![ataque_slowloris](/img/5.png)
![ataque_slowloris2](/img/6.png)

Tras activar el WAF lamentablemente al volver a realizar el ataque con slowloris, con los parámetros anteriormente definidos, el ataque sigue siendo efectivo, por lo que tras un rato de búsqueda nos topamos con un módulo llamado `mod_evasive` el cual promete proteger nuestro servidor de ataques DOS, ya que como dicen en este issue de [SpiderLabs](https://github.com/SpiderLabs/owasp-modsecurity-crs/issues/1597) el módulo no es nada eficaz contra dos medianamente complejos ni distribuidos. 

