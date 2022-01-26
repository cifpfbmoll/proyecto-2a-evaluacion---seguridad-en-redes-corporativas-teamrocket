### Indice
1. [Introducción](#introducción)
2. [Instalación](#instalación)
3. [Configuración](#conf)
    1. [Por defecto](#sudo)

## Introducción<a name="introducción"></a>

SSH es el acrónimo de secure shell. Este protocolo de administración de shell remota a diferencia de telnet u otros protocolos parecidos y caídos en desuso va encriptado. Al realizar la conexión se establece la comunicación por medio de una clave publica y otra privada. Además de ser útil para la comunicación con servidores también permite crear túneles y transferencia de archivos. En la actualidad es ampliamente utilizado.

## Instalación<a name="instalación"></a>

```bash
$apt install openssh-server
```

## Configuración<a name="conf"></a>

La mayor parte de la configuración de SSH queda plasmada en el archivo `/etc/ssh/sshd_config` la cúal debería tener únicamente permisos de lectura/escritura el usuario root:

+ Asegurarse de utilizar la Versión 2 de SSH ya que la primera tiene vulnerabilidades reconocidas y no es segura. Esto únicamente aplica a versiones OpenSSH Server anteriores a la 7.4
+ Habilitar módulo PAM para tener un mayor control de los accesos de las cuentas de usuario

    ```bash
    ...
    UsePAM yes
    ```

+ Desde el parámetro MaxStartups configuramos el número máximo de conexiones concurrentes

    ```bash
    ...
    MaxStartups 10:30:60

    #OPCIONES RECOMENDADAS EXPLICADAS

    #10 número de conexiones no auténticadas antes de rechazarlas
    #30 % de conexiones rechazadas una vez iniciadas las 10 primeras
    #60 número máximo de conexiones posibles, una vez llegado a este numero el servicio denegará nuevas conexiones
    ```

+ Configurar tiempo de comprobación de actividad o tiempo máximo de inactividad del servidor

    ```bash
    ...
    ClientAliveInterval 300 #Cada cuento se comprueba el estado
    ClientAliveCountMax 0 #Cuanto tiempo de inactividad se permite
    ```

+ Definir número máximo de intentos de inicio de sesión

    ```bash
    ...
    MaxAuthTries 3
    ```

+ Ignorar ficheros de autenticación rhost, muy inseguros

    ```bash
    ...
    IgnoreRhosts yes
    ```

+ Ignorar autenticación por host de confianza

    ```bash
    ...
    HostbasedAuthentication no
    ```

+ Inhabilitar inicio autenticación sin contraseña

    ```bash
    ...
    PermitEmptyPasswords no
    ```

+ Desactivar inicio de sesión de root

    ```bash
    ...
    PermitRootLogin no
    ```

+ Definir algoritmos de Cifrado correctos, asegurar su efectividad

    ```bash
    ...
    Ciphers aes128-ctr,aes192-ctr,aes256-ctr  
    HostKeyAlgorithms ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,ssh-rsa,ssh-dss  
    KexAlgorithms ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha256  
    MACs hmac-sha2-256,hmac-sha2-512,hmac-sha1
    ```

### Configuración sudo<a name="sudo"></a>

Su archivo de configuración está en `/etc/sudoers`, se puede acceder también con el comando `visudo`:

+ Para evitar que se dejen consolas ejecutando sudo en 2o plano:

    ```bash
    ...
    Defaults use_pty #pty al cerrar la sesión termina el proceso ejecutandose
    Defaults logfile="/var/log/sudo.log" #Definir ruta de logs
    ```

+ Definir que usuarios pueden ejecutar el que con permisos de sudo

    ```bash
    ...
    userX   127.0.0.1=(root:root)   NOPASSWD:apt update
    #user   #host=#contexto de ejecución #param:comando

    #PARÁMETROS
    #NOPASSWD
    #User_Alias
    #Host_Alias
    #Cmnd_Alias

    User_Alias PACO = yuki, paco
    Host_Alias AQUI = 0.0.0.0
    Cmnd_Alias EJE = rm -rf /*

    PACO    AQUI=(root:root)    EJE
    #el user yuki y paco pueden borrar todo el so en el contexto root
    ```

