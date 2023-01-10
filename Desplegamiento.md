# Instalaciones Desplegaments 2ºDAW


# DOCKER:
## Instalar dependencias de Docker
> Antes de comenzar la instalación de Docker, debemos asegurarnos de que nuestro servidor tenga la lista más reciente de paquetes disponibles. 
```bash
sudo apt update
 ```
 > Docker requiere instalar todas las dependencias requeridas para funcionar correctamente en Ubuntu. 
 ```bash
 sudo apt install curl gnupg  apt-transport-https ca-certificates curl software-properties-common
 ```
 ## Agregar el repositorio de Docker
 > A continuación, debemos agregar la clave GPG oficial de Docker.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
```
> Agregamos el repositorio de Docker a nuestro servidor. 
```bash
sudo add-apt-repository "deb https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
## Instalar Docker en Ubuntu
> Actualizamos paquetes.
```bash
sudo apt update
```
>Instalamos Docker con el siguiente comando:
```bash
sudo apt install docker-ce docker-ce-cli containerd.io
```
> Configurar Docker para que se inicie automáticamente.
```bash
sudo systemctl enable docker
```
## Instalar Docker-compose en Ubuntu
> El siguiente comando descargará la versión 1.26.0 y guardará el archivo ejecutable 
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
> Estableceremos los permisos correctos para que el comando docker-compose sea ejecutable:
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
> Para verificar que la instalación se realizó correctamente, puede ejecutar:
```bash
docker-compose –version
```
# Creacion HOST VIRTUAL:
### Server Ubuntu
>1. Creamos carpeta donde agregaremos el archivo index.html 
```bash
sudo mkdir /var/www/03-es-ddaw-backoffice/
```
>2. Asignamos como propietario al actualmente logueado
```bash
sudo chown -R $USER:$USER /var/www/03-es-ddaw-backoffice/
```
>3. Y le damos permisos
```bash
sudo chmod -R 755 /var/www/03-es-ddaw-backoffice/
```
>4. Creamos el archivo index.html, agregando lo que queramos.
```bash
sudo nano /var/www/03-es-ddaw-backoffice//index.html
```
>5. Creamos el archivo del host dentro de apache2
```bash
sudo nano /etc/apache2/sites-available/003-es-ddaw-backoffice.conf
```
>6. Y le añadimos estos comandos:
```bash
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName backoffice.ddaw.es
    ServerAlias www.backoffice.ddaw.es
    DocumentRoot /var/www/03-es-ddaw-backoffice
    ErrorLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-error.log
    CustomLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-access.log combined
</VirtualHost>
```
>7. Habilitamos el nuevo host
```bash
sudo a2ensite 003-es-ddaw-backoffice.conf
```
>8. Realizamos una prueba de sintaxis, comprobando que no hayan errores
```bash
sudo apache2ctl configtest
```
>9. Reiniciamos Apache2
```bash
sudo systemctl restart apache2
```
### PC anfitrión
>10. Añadimos la dirección y la ip al pc anfitrión.
```bash
sudo nano etc/hosts;
127.0.0.1       localhost
127.0.1.1       pc-batoi
192.168.1.131 web.ddaw.es
192.168.1.131 intranet.ddaw.es
192.168.1.131 example.com
192.168.1.131 backoffice.ddaw.es
```
# Módulos
> Vemos los módulos disponible
```bash
apachectl -t -D DUMP_MODULES
```
>Vemos los módulos compilados al lado del servidor
```bash
apache2 -l
```
>Directorio donde se encuentran los modulos disponibles `/usr/lib/apache2/modules`

>La carga de estos módulos se realiza con el archivo `nom_modul.load` que se encuentra en `/etc/apache2/mods-available` y su configuración en el archivo  `nom_modul.conf`.
**https://httpd.apache.org/docs/current/mod/**

>Nos dirigimos al directorio : 
```bash 
cd /etc/apache2/mods-available/
```
>y editamos el archivo :
```bash
 sudo nano status.conf
 <Location /server-status>
   SetHandler server-status
   Require local
   Require ip 192.168.56.103/24
</Location>
```
>activamos el archivo status: 
```bash
a2enmod status
```
>y nos dirigimos al navegador web y le añadimos la siguiente dirección:
**http://192.168.56.103/server-status**
  

# Activación módulo de compresión brotli
>En nuestro directorio, clonamos desde gitlab el siguiente proyecto: 
**https://gitlab.com/alecogi-edu/ddaw-ud2-a2**

>Instalamos brotli en el SO: 
```bash
sudo apt install libbrotli-dev
```
>Una vez clonado el repositorio, le instalamos el módulo.
```bash
sudo install -D mod_brotli.so /usr/lib/apache2/modules/mod_brotli.so -m 644
```
>Ahora copiamos los archivos `brotli.conf` y `brotli.load`, en el directorio `/etc/apache2/mods-available/`

>Se reinicia apache2 
```bash
sudo systemctl restart apache2
```
>El comando `wget`, nos permite realizar peticiones `HTTP` i especificar cabeceras
```bash
wget -S --header="Accept-Encoding: deflated" http :// 192.168.56.103/index.html
```
# Configuración avanzada de servidores web (seguridad)

```bash
news/
├── conf
│   └── passwords
└── html
    ├── index.html
    └── secrets
        ├── navy.html
        └── sports.html
```

>Desde `news`, creamos dos directorios nuevos, uno conf que va dentro de `news` y otro `secrets` que va dentro de `html`, donde añadimos todos los archivos .html y dandole los permisos a las carpetas del usuario.
```bash
sudo chown -R $USER:$USER /var/www/news/html/secrets/
sudo chmod -R 755 /var/www/news/conf/
```
>Después de realizar todos los puntos del anterior ejercicio hasta el 6, le  tenemos que añadir al archivo `.conf` 
```bash
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
  ServerName news.ddaw.es
  ServerAlias www.news.ddaw.es
  DocumentRoot /var/www/news/html
  ErrorLog ${APACHE_LOG_DIR}/004-es.ddaw.news-error.log
  CustomLog ${APACHE_LOG_DIR}/004-es.ddaw.news-access.log combined
  <Directory "/var/www/news/html/secrets">
    AuthType Basic
    AuthName "Restricted Files"
    AuthUserFile /var/www/news/conf/passwords
    Require valid-user
  </Directory>
</VirtualHost>
```
```bash
sudo a2ensite 004-es.ddaw.news.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```
>Creamos los usuarios que pueden entrar en la web

>El primero lleva `-c`, porque es el que crea el archivo.
```bash
$ htpasswd -c /var/www/news/conf/passwords reader
$ htpasswd /var/www/news/conf/passwords sports
$ htpasswd /var/www/news/conf/passwords national
```
>Vamos a hosts de la maquina anfitriona y le añadimos el servidor.
```bash
sudo nano /etc/hosts
```
```bash
127.0.0.1       localhost
127.0.1.1       pc-batoi
192.168.56.103 news.ddaw.es
```
>Descentralizar la configuración de htaccess

```bash
news/
├── conf
│   └── passwords
└── html
    ├── index.html
    └── secrets
        ├── .htaccess
        ├── navy.html
        └── sports.html
```
>Vamos a crear un fichero oculto `.htaccess` dentro de `secrets`.

>Y le añadimos las directivas que tenía el `.conf`.
```bash
AuthType Basic
AuthName "Restricted Files"
AuthUserFile /var/www/01-es-ddaw-intranet/conf/passwords
Require valid-user
```
>Y en el `.conf`:
```bash
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName intranet.ddaw.es
    ServerAlias www.intranet.ddaw.es
    DocumentRoot /var/www/01-es-ddaw-intranet
    ErrorLog ${APACHE_LOG_DIR}/001-es-ddaw-intranet-error.log
    CustomLog ${APACHE_LOG_DIR}/001-es-ddaw-intranet-access.log combined
        <Directory "/var/www/01-es-ddaw-intranet/html/secrets">
            AllowOverride AuthConfig
        </Directory>
</VirtualHost>
```
>Una vez realizado los cambios en los dos archivos, volvemos a reiniciar apache2
```bash
sudo a2ensite 001-es-ddaw-intranet.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```
>Ahora vamos a crear una contraseña codificada, antes de crear los usuarios: 
```bash
openssl rand -base64 14
```
>Del usuario “admin1” será `5laZ/GrGFRppZEssQO4`

>Y la del usuario “developer1” será `nqhPZBiAA9xCzQ02Xrc`

>y creamos los usuarios:

>El primero lleva `-c`, porque es el que crea el archivo.
```bash
$ htpasswd -c /var/www/01-es-ddaw-intranet/conf/passwords admin1
$ htpasswd /var/www/01-es-ddaw-intranet/conf/passwords developer1
```
>Y ya tendremos la configuración de `htaccess`  descentralizada.

# Parte que cambiamos a Digest el esquema de autentificación.
>Habilitar el módulo `auth_digest`
```bash
sudo a2enmod auth_digest
```
>y reiniciamos el servidor…
```bash
sudo systemctl restart apache2
```
>Creamos el nuevo archivo de passwords llamándolo `passwordsDigest`
```bash
sudo htdigest -c /var/www/01-es-ddaw-intranet/conf/passwordsDigest restringido mika
```
>y le añadimos las contraseñas

>Vamos al archivo .htaccess y lo modificamos:
```bash
AuthType Digest
AuthName "restringido"
AuthUserFile /var/www/01-es-ddaw-intranet/conf/passwordsDigest
Require valid-user
```
> Y a tendremos cambiado la configuración en Digest

# Configuración SSL de servidor apache2: 
## Certificados de servidor auto-firmados

>1. Actualizamos el equipo:
```bash
sudo apt update; sudo apt upgrade;
```

>2. Instalamos las herramientas de openssl
```bash
sudo apt install openssl libssl-dev;
```
>3. Activamos el modulo de ssl
```bash
sudo a2enmod ssl;
```
>4. Reiniciamos apache2
```bash
sudo systemctl restart apache2;
```

## Creació del certificat de servidor (openssl)

>1. Creamos carpeta en el directorio principal.
```bash
cd;
```
>2. Creamos la carpeta  `intranet-ssl-cert`
```bash
sudo mkdir intranet-ssl-cert;
```
>3. Entramos dentro de ella
```bash
cd intranet-ssl-cert;
```
>4. Creamos el certificado y los archivos de este
```bash
sudo openssl req -new -x509 -nodes -days 365 -newkey rsa:2048 -out server.crt -keyout server.key
```
>Y rellenamos los campos que nos pide:
```bash
Country Name (2 letter code) [XX]:ES
State or Province Name (full name) []: Alacant
Locality Name (eg, city) [Default City]: Alcoi
Organization Name (eg, company) [Default Company Ltd]: Example Inc
Organizational Unit Name (eg, section) []: Example Dept
Common Name (eg, your name or your server's hostname) []: intranet.ddaw.es
Email Address []: webmaster@intranet.ddaw.es
```
>5. Creamos una carpeta con el nombre del dominio en:
```bash
 sudo mkdir /etc/ssl/intranet.ddaw.es
```
>6. Le añadimos los archivos creados
```bash
sudo cp server.crt server.key /etc/ssl/intranet.ddaw.es/
```

## Configuración del módulo SSL
>1. Añadimos directiva para escuchar al nuevo `puerto 443`
```bash
sudo nano /etc/apache2/ ports.conf 
```
```bash 
<IfModule ssl_module>
	Listen 443
</IfModule>
```
>2. Anadimos las directivas dentro del archivo del dominio
```bash
sudo nano /etc/apache2/sites-available/001-es-ddaw-intranet.conf
```
```bash
<VirtualHost *:443>
	SSLEngine On
	SSLCertificateFile /path/to/server.crt
	SSLCertificateKeyFile path/to/server.key
</VirtualHost>
```
>Pero tambien le añadimos las directivas del `puerto 80`
```bash
<VirtualHost *:443>
  ServerAdmin webmaster@localhost
  ServerName intranet.ddaw.es
   ServerAlias www.intranet.ddaw.es
   DocumentRoot /var/www/01-es-ddaw-intranet/html
   ErrorLog ${APACHE_LOG_DIR}/001-es-ddaw-intranet-error.log
  CustomLog ${APACHE_LOG_DIR}/001-es-ddaw-intranet-access.log combined
    <Directory "/var/www/01-es-ddaw-intranet/html/secrets">
        AllowOverride AuthConfig
    </Directory>
  SSLEngine On
  SSLCertificateFile /etc/ssl/intranet.ddaw.es/server.crt
  SSLCertificateKeyFile /etc/ssl/intranet.ddaw.es/server.key
</VirtualHost>
``` 
>3. Reiniciamos el servidor
```bash
sudo systemctl restart apache2;
```
>Y vemos como nos dice que el sitio que entramos, no es seguro.

>Para redirigir permanentemente de HTTP a HTTPS, le añadimos esta linea:
```bash
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName backoffice.ddaw.es
    ServerAlias www.backoffice.ddaw.es
    DocumentRoot /var/www/03-es-ddaw-backoffice
    //linea
    Redirect permanent / https://backoffice.ddaw.es
    //
    ErrorLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-error.log
    CustomLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-access.log combined
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName backoffice.ddaw.es
    ServerAlias www.backoffice.ddaw.es
    DocumentRoot /var/www/03-es-ddaw-backoffice
    SSLEngine On
    SSLCertificateFile /etc/ssl/backoffice.ddaw.es/server.crt
    SSLCertificateKeyFile /etc/ssl/backoffice.ddaw.es/server.key
    ErrorLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-error.log
    CustomLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-access.log combined
</VirtualHost>
```
> también podemos añadirle estas líneas...
```bash
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName backoffice.ddaw.es
    ServerAlias www.backoffice.ddaw.es
    DocumentRoot /var/www/03-es-ddaw-backoffice
    ErrorLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-error.log
    CustomLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-access.log combined
  //líneas
    RewriteEngine On
    RewriteCond %{HTTPS} !=on
    RewriteRule ^/?(.*) https://%{SERVER_NAME}/$1 [R=301,L]
  //
</VirtualHost>

<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName backoffice.ddaw.es
    ServerAlias www.backoffice.ddaw.es
    DocumentRoot /var/www/03-es-ddaw-backoffice
    SSLEngine On
    SSLCertificateFile /etc/ssl/backoffice.ddaw.es/server.crt
    SSLCertificateKeyFile /etc/ssl/backoffice.ddaw.es/server.key
    ErrorLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-error.log
    CustomLog ${APACHE_LOG_DIR}/003-es-ddaw-backoffice-access.log combined
</VirtualHost>
```

# Instalación de FTP
>Instalamos el paquete `inetutils`
```bash
sudo apt install inetutils-tools o sudo apt install ftp
```
> La siguiente tabla, muestra los comandos más utilizados

| Orden  | Función |
| ------------- |:-------------:|
| **ftp** | Accede al intérprete de comandos `ftp`. |
| **ftp + url** | Establece una conexión ftp a un sistema remoto. |
| **open** | Inicia sesión en el sistema remoto desde el intérprete de comandos |
| **close** | Cierra la sesión del sistema remoto y vuelve al intérprete de comandos. |
| **bye** | Sale del intérprete de comandos `ftp`. |
| **help** | Muestra todos los comandos `ftp` |
| **reset** | Vuelve a sincronizar la secuenciación de respuesta de comando con el servidor ftp remoto. |
| **ls** | Muestra los contenidos del directorio de trabajo remoto. |
| **pwd** | Muestra el nombre del directorio de trabajo remoto. |
| **cd** | Cambia el directorio de trabajo remoto. |
| **lcd** | Cambia el directorio de trabajo local. |
| **mkdir** | Crea un directorio en el sistema remoto. |
| **rmdir** | Elimina un directorio en el sistema remoto. |
| **get, mget** | Copia un archivo (o varios archivos) del directorio de trabajo remoto al directorio de trabajo local. |
| **put, mput** | Copia un archivo (o varios archivos) del directorio de trabajo local al directorio de trabajo remoto. |
| **delete, mdelete** | Elimina un archivo (o varios archivos) del directorio de trabajo remoto. |
| **l + comando** | Permite ejecutar comandos de forma local, como: `lls`, `lcd` |

> Una vez que hemos instalado FTP, y ya tenemos una cuenta en el hosting para webs, iniciamos el contacto con esta:

>podemos realizarlo de dos maneras:

```bash
ftp
```
>y dentro
```bash
 open files.000webhost.com
 ```
 > o todo junto desde linea de comandos de la terminal de linux
 ```bash
ftp files.000webhost.com 
```
>una vez conectados con el `hostig`, vamos a la carpeta `public_html` y le subimos nuestros archivos desde el cliente, en el que también tendremos que estar en el directorio donde se encuentran los ficheros.

>Desde el directorio web
```bash
put index.html
```
>, y desde el directorio donde se encuentran las imagenes
```bash
mput *.jpeg 
```
> con este comando, lo que hacemos es subir todos los archivos con extensión `jpeg`

## Códigos de estado ftp
### 1xx - Respuesta preliminar positiva
>Estos códigos de estado indican que una acción se ha iniciado correctamente, pero el cliente espera otra respuesta antes de continuar con un nuevo comando.

* 110 - Respuesta del marcador de reinicio.
* 120 - Servicio listo en nnn minutos.
* 125 - Conexión de datos ya abierta; inicio de la transferencia.
* 150 - El estado del archivo está bien; a punto de abrir la conexión de datos.
### 2xx - Respuesta de finalización positiva
>Una acción se ha completado correctamente. El cliente puede ejecutar un nuevo comando.

* 200 - Comando correcto.
* 202 - Comando no implementado, superfluo en este sitio.
* 211 - Estado del sistema o respuesta de ayuda del sistema.
* 212: estado del directorio.
* 213: estado del archivo.
* 214 - Mensaje de ayuda.
* 215 - Tipo de sistema NAME, donde NAME es un nombre oficial del sistema de la lista en el documento Números asignados.
* 220: servicio listo para el nuevo usuario.
* 221 - Conexión de control de cierre del servicio. Se ha cerrado la sesión si procede.
* 225 - Conexión de datos abierta; no hay transferencia en curso.
* 226 - Cierre de la conexión de datos. Acción de archivo solicitada correcta (por ejemplo, transferencia de archivos o anulación de archivos).
* 227 - Entrar en modo pasivo (h1,h2,h3,h4,p1,p2).
* 229 - Se ha introducido el modo pasivo extendido.
* 230 - Usuario que ha iniciado sesión, continúe.
* 232- El usuario ha iniciado sesión, autorizado por el intercambio de datos de seguridad.
* 234 - Intercambio de datos de seguridad completado.
* 235- El intercambio de datos de seguridad se completó correctamente.
* 250 - Acción de archivo solicitada bien, completada.
* 257 - "PATHNAME" creado.
### 3xx - Respuesta intermedia positiva
>El comando se realizó correctamente, pero el servidor necesita información adicional del cliente para completar el procesamiento de la solicitud.

* 331 - Nombre de usuario bien, necesita contraseña.
* 332 - Necesita una cuenta para el inicio de sesión.
* 334 - Mecanismo de seguridad solicitado correcto.
* 335- Los datos de seguridad son aceptables. Se requieren más datos para completar el intercambio de datos de seguridad.
* 336 - Nombre de usuario bien, necesita contraseña.
* 350 - Acción de archivo solicitada pendiente de más información.
### 4xx - Respuesta de finalización negativa transitoria
>El comando no se realizó correctamente, pero el error es temporal. Si el cliente reintenta el comando, puede realizarse correctamente.

* 421 - Servicio no disponible, conexión de control de cierre. Puede ser una respuesta a cualquier comando si el servicio sabe que debe apagarse.
* 425: no se puede abrir la conexión de datos.
* 426 - Conexión cerrada; transferencia anulada.
* 431: necesita algún recurso no disponible para procesar la seguridad.
* 450: no se ha realizado la acción de archivo solicitada. Archivo no disponible (por ejemplo, archivo ocupado).
* 451 - Acción solicitada anulada. Error local en el procesamiento.
* 452 - Acción solicitada no adoptada. Espacio de almacenamiento insuficiente en el sistema.
### 5xx - Respuesta de finalización negativa permanente
>El comando no se realizó correctamente y el error es permanente. Si el cliente reintenta el comando, recibe el mismo error.

* 500: error de sintaxis, comando no reconocido. Esto puede incluir errores como la línea de comandos demasiado largo.
* 501: error de sintaxis en parámetros o argumentos.
* 502: comando no implementado.
* 503- Secuencia incorrecta de comandos.
* 504: comando no implementado para ese parámetro.
* 521- No se puede abrir la conexión de datos con esta configuración prot.
* 522: el servidor no admite el protocolo de red solicitado.
* 530- No ha iniciado sesión.
* 532- Necesita una cuenta para almacenar archivos.
* 533: nivel de protección de comandos denegado por motivos de directiva.
* 534- Solicitud denegada por motivos de directiva.
* 535: comprobación de seguridad errónea (hash, secuencia, etc.).
* 536- Nivel PROT solicitado no compatible con el mecanismo.
* 537: nivel de protección de comandos no compatible con el mecanismo de seguridad.
* 550 - Acción solicitada no adoptada. Archivo no disponible (por ejemplo, archivo no encontrado o sin acceso).
* 551 - Acción solicitada anulada: Tipo de página desconocido.
* 552: acción de archivo solicitada anulada. Se ha superado la asignación de almacenamiento (para el directorio o conjunto de datos actual).
* 553 - Acción solicitada no adoptada. No se permite el nombre de archivo.
### 6xx : respuesta protegida
>Estos códigos de estado indican una respuesta protegida desde FTP.

* 631 - Respuesta protegida por integridad.
* 632 - Respuesta protegida de confidencialidad e integridad.
* 633 - Respuesta protegida por confidencialidad.

# Instalación del servidor FTP/S
> Instalamos desde línea de comandos
```bash
sudo apt install vsftpd;
```
> Después de la instalación se creará automáticamente un usuario llamado `FTP`, y la carpeta local del que estará en `/srv/ftp` y que hará la funciónde usuario anónimo
```bash
cat /etc/passwd
```
>**ftp: x :108:113:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin**

>Para gestionar el demonio del `vsftpd` utilizaremos el mando correspondiente `systemctl` o su `service`

```bash
systemctl [start|stop|restart] vsftpd
service vsftp [start|stop|restart]
```
## Configuración del servidor
>En primer lugar, crearemos una copia de
seguridad del archivo principal `vsftpd.conf`
```bash
$ sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.back
```

>Como en los servicios configurados en las últimas unidades, las líneas de este archivo sigue el siguiente patrón:

>`directiva=valor`

|Directiva|Valor|
| ------------- |:-------------:|
|**listen=YES** | Para que se ejecute vsftpd en modo independiente.  |
|**anonymous_enable=NO**|No permitimos que se conecten usuarios anónimos.|
|**local_enable=YES**|Permitimos que los usuario locales se puedan conectar.|
|**write_enable=YES**|Permitimos poder hacer modificaciones.|
|**dirmessage_enable=YES**|Muestra un mensaje cada vez que un usuario entra en un directorio.|
|**xferlog_enable=YES**|Registra las conexiones y la información de transferencia, por defecto en `/var/log/vsftpd.log`|
|**connect_from_port_20=YES**|Se permite que el servidor vsftpd abra el puerto 20, para ponerse a la escucha de peticiones.|
|**ftpd_banner=Bienvenidos al ftp de Redes de Area Local.**|Mensaje de bienvenida al conectarse mediante un cliente ftp|
|**chroot_local_user=NO**|Permitimos a los usuarios locales que puedan salir de su directorio.|
|**chroot_list_enable=YES**|Los usuarios locales que se encuentren en el fichero indicado por `chroot_list_file` estarán enjaulados en su directorio.|
|**chroot_list_file=/etc/vsftpd.chroot_list**|Especifica el fichero que contiene los usuarios a enjaular.|
|**secure_chroot_dir=/var/run/vsftpd**|Es un directorio usado como una jaula segura chroot.|
|**pasv_min_port=40000**|Puerto mínimo que puede ser asignado para la conexión de datos|
|**pasv_max_port=50000**|Puerto máximo que puede ser asignado a la conexión de datos|
|**local_umask = 022**|Establece la máscara aplicable para los archivos cargados|
|**xferlog_enable=YES**|Activa/Desactiva el registro de actividad de las cargas y descargas|
|**xferlog_file=/var/log/vsftpd.log**|Localización del archivo de `logs`|
|**log_ftp_protocol=YES**|Activa/Desactiva el log para el protocolo `vsftpd`|
|**data_connection_timeout = 120**|Tiempo máximo que el servidor espera cuando una transferencia no progresa|
|**idle_session_timeout = 600**|Tiempo máximo que se concede a un usuario remoto que no está activo| 

>Con la ejecución del siguiente mando nos podremos ver la escritura del log en tiempo
real
```bash
$ tail -f /var/log/vsftpd.log
```

|Tipo|Permisos_ defecto|_Máscara_(umask)|Permisos_obtenidos|
| ------------- | ------------- | ------------- |------------- |
|Archivo|666|022|(666-022)→644|
|Directorio|777|022|(777-022)→755|   
 
 
>El siguiente paso será abrir los correspondientes puertos en el firewall
```bash
sudo ufw allow20/tcp $
sudo ufw allow21/tcp $
sudo ufw allow990/tcp
sudo ufw allow40000:50000/tcp
```
>No está permitido que los usuarios
escriban en la carpeta raíz, donde hemos "enjaulado" el usuario. Para solucionarlo, crearemos una carpeta FTP en cada home del usuario.
```bash
sudo mkdir /home/{usuario}/ftp.
```
|Directiva|Valor|
| ------------- |:-------------:|
|**user_sub_token=$USER**|Establecemos que pueda utilizarse el contenido de `$USER` como parte del directorio de usuario|
|**local_root=/home/$USER/ftp**|Establecemos un nuevo directorio raíz para el usuario|
|**allow_writeable_chroot=YES**|Permitimos que los usuarios local puedan escribir en el directorio raíz|

# Instalación de NGinx
##  Instalar Nginx
```bash
sudo apt update
sudo apt install nginx
```
## Configurar el Firewall
```bash
sudo ufw app list
```
```bash
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
  ```
>`Nginx Full`: Este perfil abre tanto el puerto 80 (tráfico web normal, no cifrado) como el puerto 443 (tráfico TLS/SSL cifrado)

>`Nginx HTTP`: Este perfil solamente abre el puerto 80 (tráfico web normal, no cifrado)

>`Nginx HTTPS`: Este perfil solamente abre el puerto 443 (tráfico TLS/SSL cifrado)

>Lo habilitamos con el comando: 
```bash
sudo ufw allow 'Nginx HTTP'
```
>y vemos el estado del firewall
```bash
sudo ufw status
Output
Status: active
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```
## Verificar su servidor web
```bash
systemctl status nginx
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2018-04-20 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```
## Gestionar el proceso de Nginx
>Para detener su servidor web, ingrese:
```bash
sudo systemctl stop nginx
```
>Para iniciar el servidor web una vez que se haya detenido, ingrese:
```bash
sudo systemctl start nginx
```
>Para detener y luego volver a iniciar el servicio, ingrese:
```bash
sudo systemctl restart nginx
```
>Recargar `Nginx` sin perder las conexiones. Para hacerlo, ingrese:
```bash
sudo systemctl reload nginx
```
>Para volver a habilitar el servicio para que empiece tras la iniciación, puede ingresar:
```bash
sudo systemctl enable nginx
```
## Configurar los bloques del servidor
>Cree el directorio para example.com como se indica a continuación, usando el indicador `-p` para crear cualquier directorio matriz que pueda requerirse:
```bash
sudo mkdir -p /var/www/example.com/html
```
>Posteriormente, asigne la titularidad del directorio con la variable de entorno `$USER`:
```bash
sudo chown -R $USER:$USER /var/www/example.com/html
```
>Si no ha modificado su valor de `umask`, los permisos de sus roots web deberían ser los correctos, pero puede verificarlo ingresando:
```bash
sudo chmod -R 755 /var/www/example.com
```
>Luego, cree una página `index.html` como ejemplo utilizando nano o su editor preferido:
```bash
nano /var/www/example.com/html/index.html
```
>Adentro, agregue el siguiente HTML como ejemplo:
```html
<html>
    <head>
        <title>Welcome to Example.com!</title>
    </head>
    <body>
        <h1>Success!  The example.com server block is working!</h1>
    </body>
</html>
```
>En vez de modificar el archivo de configuración predeterminado directamente, vamos a hacer uno nuevo en `/etc/nginx/sites-available/example.com`:
```bash
sudo nano /etc/nginx/sites-available/example.com
```
>Y le añadimos la siguiente configuración:
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
>Después, vamos a habilitar el archivo creando un enlace desde el mismo al directorio sites-enabled (habilitado para sitios), el cual Nginx usa para leer durante el inicio:
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```
>Para evitar un posible problema de memoria de hash bucket, abra el archivo:
```bash
sudo nano /etc/nginx/nginx.conf
```
>Busque la directiva `server_names_hash_bucket_size` y quite el símbolo `#` para descomentar la línea:
```
...
http {
    ...
    server_names_hash_bucket_size 64;
    ...
}
...
```
>Haga una prueba para asegurarse de que no haya errores de sintaxis:
```bash
sudo nginx -t
```
>Si no hay ningún problema, reinicie `Nginx` para habilitar sus cambios:
```bash
sudo systemctl restart nginx
```
## Familiarizarse con archivos y directorios importantes de Nginx
### Contenido
>`/var/www/html:` El contenido web real, y puede modificarse alterando los archivos de configuración de Nginx.
### Configuración del servidor
>`/etc/nginx`: El directorio de configuración de Nginx.

>`/etc/nginx/nginx.conf`: El archivo de configuración de Nginx principal.

>`/etc/nginx/sites-available/`: El directorio donde se pueden almacenar los bloques del servidor por sitio. Nginx no usará los archivos de configuración que estén en este directorio a menos que estén vinculados al directorio sites-enabled.

>`/etc/nginx/sites-enabled/`: El directorio donde se almacenan los bloques del servidor por sitio habilitados.

>`/etc/nginx/snippets`: Este directorio contiene fragmentos de configuración que se pueden incluir en cualquier otro sitio de la configuración de Nginx.
### Registros del servidor
>`/var/log/nginx/access.log`: Se registra cada solicitud a su servidor web.

>`/var/log/nginx/error.log`: Todo error de Nginx se registrará en este registro.

# Instalación del servidor de aplicaciones php-fpm
> Debido a que `NGinx` no puede ejecutarcódigo `PHP` de forma nativa, deberemos hacer uso de un servicio independiente, en este caso hemos elegido un servidor `php-fpm`

## Instalación del servicio php-fpm
```bash
sudo apt install php-fpm
```
> Para Ubuntu 22.04 Server, se instalará la
versión `PHP 8.1`. Este servicio se gestiona de forma independiente al del servidor web,
mediante el comando `systemctl` o su correspondiente `service`.
```bash
systemctl [enable|disable|start|stop|restart]php8.1-fpm
service php8.1-fpm [start|stop|restart]
```
## Estructura de archivos.
>La instalación básica, crea un pool de conexiones, cuyo archivo de configuración
se encuentra: `/etc/php/{ version-php }/fpm/pool.d/www.conf`.

|Parámetros|Descripción|
| ------------- |:-------------:|
|**identificador** [www]|identificador/nombre delpool, Si tenemos varios,cada uno debe tener un número diferente|
|**user, group** user = www-fecha group = www-fecha|Usuario y grupo con el que se ejecutaron los procesos de Apache (Determinará los permisos que debería tener cada DocumentRoot|
|**Listan**|Debemos indicar el socket unix o el socket TCP que atenderá a las peticiones.| 
|`Socket unix`|`listan = /var/run/php/php8.1-fpm.sock`
|
|`socket tcp`|`listan = 127.0.0.1:9000`|
|`socket tcp` en cualquier interfaz de red local en la máquina|`listan = 9000`|
|pm|Directivas para la gestión de procesos|
|`pm.max_children = 64`|se establece un máximo de 64 procesos activos|
|`pm.max_requests = 500`|un máximo de 500 peticiones concurrentes|
|`pm.status_path = /status`|se habilita una página de monitorización del servicio php-fpm|
|acceso|Directivas para la gestión del archivo de log para las peticiones que atiende al servidor php-fpm|
|access.log = `/var/log/$pool.access.log`||
|access.format= `"%R - %o %t \"%m %r%Q %q\"`||

## Definición nuevo `pool`

> Ejecutamos el comando, para hacer una copia del archivo `www.conf`, y así poder configurar nuestro nuevo `pool` 
```bash
sudo cp /etc/php/8.1/fpm/pool.d/www.conf /etc/php/8.1/fpm/pool.d/001-ddaw-es.conf
 ```
 Dentro de este archivo, debemos configurar con los parametros que queremos cambiar.
 ```
 #Identificador del pool
[001-es-ddaw]
#Usuari i grup amb el qual es llegiran i executaran els documents PHP
user = www-data
group = www-data
#Socket de connexió
listen = /var/run/php/php8.1-fpm-001-es-ddaw.sock
 ```
 > #0969DA Una vez terminado la edición del archivo, haremos un restart al servicio...
 ```bash
 sudo service php8.1-fpm restart
 ```

 ## Configuración del server block.
 > Primero editaremos el archivo de la configuracion `NGinx`, que esta en `/etc/NGinx/sites-available/001-com-example`, y le añadimos dentro del server, si establecemos la conexión mediante `Unix Socket` o `TCP Socket`
 ```
 server {
    llisten 80;
    root /var/www/html;
    index index.php index.html;
    server_name example.com;
    # Definim un nou bloc de configuració que s'aplicarà quan
    # s'accedisca a qualsevol fitxer amb extensió .PHP
    location ~ \.php$ {
      #Incloem configuració específica per al servei
      include snippets/fastcgi-php.conf;
      #Establim la comunicació mitjançant Unix Socket
      #fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
      #Establim la comunicació mitjançant TCP Socket
      #fastcgi_pass 127.0.0.1:9000;
    }
  }
 ```
 > Vemos que tenemos que descomentar para Unix `fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;` o para TCP `fastcgi_pass 127.0.0.1:9000;`

> Para prbar el servicio, el interprete de `PHP` nos proporciona una función nativa `phpinfo()`, proporcionando información sobre el estado de las opciones de compilación.

>Generamos un index.php y le añadimos el siguiente comando `php` dentro de él.
```php
<?php phpinfo(); ?>
```

# Instalación y configuración servidor MYSQL

> Vamos a instalar un servido MYSQL y que interactue con el servidor de aplicaciones.

>Instalamos MYSQL
```bash
 sudo apt install mysql-server
```
> Como en todos los servicios que hemos instalado, tenemos los comandos para gestionar el servicio
```bash
systemctl [enable|disable|start|stop|restart] mysql 
service mysql [start|stop|restart]
```
>Para entrar dentro del servidor, lo haremos como usuario `root`, ejecutando el comando:
```bash
sudo mysql
```
## Estructura de directorios:
|Archivo|Descripción|
| ------------- |:-------------:|
|**/etc/mysql/conf.d/mysqld.cnf**|Contiene las configuraciones globales de MYSQL|
|**/etc/mysql/mysql.conf.d/mysqld.cnf**|Con el fin de mantener retrocompatibilidad hacia atrás, encontramos el mismo archivo con las configuraciones por defecto en el path|
|**/etc/mysql/conf.d/mysqldump.cnf**|Permite establecer configuraciones globales para la herramienta| 

## Directivas del archivo `mysqld.cnf`:
|Directiva|Descripción|
| ------------- |:-------------:|
|**user `user=mysql`** |Establece el usuario del S.O. con el que se ejecutará el proceso de mysql.|
|**port `port=3306`**|Puerto por defecto en el que el servidor se mantendrá a la escucha|
|**bind-address `bind-address=127.0.0.1`**|Establece la interfaz de red para la que atenderá las peticiones el servidor mysql.|
|**log_error `log_error=/var/log/mysql/error.log`**|Archivo de log por defecto del servicio|
|**slow_query_log `slow_query_log=1`**|Activación / desactivación del registro de consultas pesadas que se están llevando a cabo en el servidor|  

## Creación de de base de datos y usuarios
> Para crear la base de datos, cuando estemos dentro del servidor `MYSQL` le ejecutamos el siguiente comando:
```sql
mysql> CREATE DATABASE myDataBase;
```
>Para crear el usuario utilizamos el comando:
```sql
mysql> CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
```
- <span style="color:red">**user:**</span> Usuarios que utilizaremos para conectarnos con el servidor
- <span style="color:red">**localhost:**</span> Ip desde la que este usuario podrá conectarse al servidor. Si utilizamos `%`, este usuario podrá conectarse desde cualquier `IP`
- <span style="color:red">**password:**</span>  Password que utilizaremos para autenticarnos.
  
> Podemos hacer uso de la herramienta openssl para generar un password seguro
```bash
openssl rand -base64 32
```
>Podemos consultar a los usuarios de mysql mediante una consulta en el diccionario de datos del sistema.
```sql
mysql> SELECT user,plugin,host FROM mysql.user;
```
>Para asignar permisos a nuestro usuario, ejecutamos el siguiente comando en `MYSQL`
```sql
mysql> GRANT ALL PRIVILEGES ON myDataBase.* TO 'user'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> exit();
```
> Comando y opciones para conectarnos mediante usuario/password.
```sql
mysql --user=app1-prod --password crm-db
Escribir password -> $us3rdd4au$
```
> Directiva nos permite especificar una política de expiración de contraseñas
```
expire_password_interval=180
```
> Las interfaces que se mantendrá a la escucha en el servidor, si no se especifica la directiva, son:
```
 0.0.0.0 (IPv4) i [::] (IPv6)
```
> Cómo establecemos únicamente permisos DML para un usuario
```sql
GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON myDatabrostisca.* TO 'user’@’localhost';
```
> Qué funcionalidad nos proporciona cada una de las librerias nativas cargadas:

> Las librerías nativas cargadas proporcionan funcionalidades para el acceso a datos, consultas, inserción, modificación, eliminación... También ofrecen funciones para la gestión de conexiones y resultados de errores relacionados con las consultas SQL, así como funciones de codificación y seguridad.

> La función realizan los archivos `.ini` que se han creado, son para configurar las librerias nativas cargadas

> Comando para importar archivo .sql a la base de datos:
```sql
sudo mysql -u'root' < db_backup/crm_db.sql 
```


## Instalación del modulo `PHP-MYSQL`
```bash
sudo apt install php-mysql;
```
> Archivo configuración de la base de datos
```bash
/var/www/app-ddaw-cipfpbatoi-es/html/ddaw-ud4-a1/config/database-params.php
```
```php
return [
         "host" => "127.0.0.1",
         "user" => "admin_db",
         "password" => "1234",
         "database" => "crm_db"
   ];
```
> Creación de usuario administrados, con accesibilidad desde fuera
```sql
CREATE USER 'admin_db'@'%' IDENTIFIED BY '1234';
```
> Y le damos todos los privilegios
```sql
GRANT ALL PRIVILEGES ON crm_db.* TO 'admin_db'@'%';
```

## Instalación de la aplicación MysqlWorkBench
> Descargaremos la versión de linux ubuntu desde la web oficial desde
[Este link](https://dev.mysql.com/downloads/workbench/)
> Una ves desde descargas utilizamos el comando de instalación
```bash
sudo dpkg -i mysql-workbench-community-dbgsym_8.0.31-lubuntu22.10_amd64.deb
```
> Despues habilitamos la conexiones desde fuera en el archivo...
```bash
/etc/mysql/mysql.conf.d/mysqld.cnf 
```
```bash
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address		= 0.0.0.0
mysqlx-bind-address	= 127.0.0.1
```
> Y cambiamos el usuario
```bash
# * Basic Settings
#
user		= mcastro
# pid-file	= /var/run/mysqld/mysqld.pid
# socket	= /var/run/mysqld/mysqld.sock
# port		= 3306
# datadir	= /var/lib/mysql
```
> Y despues creamos nuevas conexiones dentro de la aplicación

# Configuración del entorno de ejecución de la aplicación Laravel Framework.


# Crear carpeta y dar permisos

```bash
sudo mkdir /var/www/ddaw-example/html -p
sudo chown $USER:$USER /var/www/ddaw-example/html
sudo chmod 755 /var/www/ddaw-example/htm
```
<br>

# Clonamos el repositorio y editamos el .env

```json
APP_ENV=production
APP_DEBUG=false
APP_KEY=b809vCwvtawRbsG0BmP1tWgnlXQypSKf
APP_URL=http://example.com
DB_HOST=127.0.0.1
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=password
```

<br>


# Damos permisos bootstrap

``` bash
cd myNewApp;
sudo chgrp -R www-data storage bootstrap/cache;
sudo chmod -R ug+rwx storage bootstrap/cache;
```
<br>


# Creamos Nginx File

``` json
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    root /var/www/ddaw-example/html/myNewApp/public/;
    index index.htm index.html index.php;
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

> IMPORTANTE: Acordaros de que sea la carpeta public del proyecto y de crear el socket por si esta ocupado

<br>

# Crear el enlaze

```bash
sudo ln -s /etc/nginx/sites-available/nombreArchivo /etc/nginx/sites-enabled
```
<br>

# Reinciar servicios

<br>

# Editar el archivo hosts del local