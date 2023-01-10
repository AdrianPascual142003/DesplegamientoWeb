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