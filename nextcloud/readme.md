# Instalación Nextcloud 28.01 en VM Ubuntu 22.04

Aunque la implementación de Nextcloud vía Docker se haya popularizado bastante, os dejo una guía para instalar esta aplicación en una VM.

[![Nextcloud](https://cdn.bayton.org/uploads/2017/05/NC-docs_Banner.png "Nextcloud")](https://nextcloud.com/ "Nextcloud")

#### 1. Preparación VM Ubuntu 22.04

Instalamos Apache, y MySQL

    sudo apt update
    sudo apt install apache2
    sudo apt install mysql-server

Y securizamos la instancia:

`sudo mysql_secure_installation`

Nos preguntará si queremos establecer un componente de contraseñas seguras, eliminar acceso externo para el usuario root, etc. 

#### 2. Conexión y configuración Base de Datos

`sudo mysql`

Creamos una base de datos denominada "nextcloud", con usuario "nextcloud" y la contraseña que queramos, y daremos privilegios a dicho usuario sobre la base de datos:

    create database nextcloud;
    create user 'nextcloud'@'localhost' identified by 'mipassword';
    grant all privileges on nextcloud.* to 'nextcloud'@'localhost';
    flush privileges;
    quit

#### 3. Instalación PHP y componentes necesarios

A continuación instalaremos PHP 8.3 y los componentes necesarios. 

> :warning: Esta es la versión que instalamos a fecha de Enero de 2024. Dependiendo de cuando leas este tutorial, puede haber una versión más actualizada. Compruébalo antes de proceder

    sudo apt-get install ca-certificates apt-transport-https software-properties-common
    sudo add-apt-repository ppa:ondrej/php
    sudo apt-get update
    sudo apt install php8.3

Y comprobaremos si se ha instalado correctamente:

`php8.3 --version`

Deberíamos ver un output como este:

    PHP 8.3.1 (cli) (built: Dec 21 2023 20:12:13) (NTS)
    Copyright (c) The PHP Group
    Zend Engine v4.3.1, Copyright (c) Zend Technologies
        with Zend OPcache v8.3.1, Copyright (c), by Zend Technologies

Tras comprobarlo, instalamos los componentes necesarios:

    sudo apt install php8.3-gd php8.3-mysql php8.3-curl php8.3-mbstring
    sudo apt install php8.3-intl php8.3-gmp php8.3-bcmath php8.3-xml
    sudo apt install php8.3-zip php8.3-bz2 php-imagick php-apcu

#### 4. Instalación Nextcloud

Procedemos a la descarga del .zip y su extracción

    wget https://download.nextcloud.com/server/releases/latest.zip
    sudo apt install unzip
    sudo unzip latest.zip -d /var/www
    cd /var/www
    sudo chown -R www-data:www-data nextcloud/

y configuramos Apache:

    sudo a2enmod headers env dir mime
    cd /etc/apache2/sites-available/
    sudo vi 000-default.conf

editamos el documento "000-default.conf", dejándolo de la siguiente manera:

    <VirtualHost *:80>
            #ServerAdmin webmaster@localhost
            DocumentRoot /var/www/nextcloud
            <Directory /var/www/nextcloud/>
                    Require all granted
                    AllowOverride All
                    Options FollowSymLinks Multiviews
                    <Ifmodule mod_dav.c>
                            Dav off
                    </IfModule>
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

Si quisiésemos acceder desde el exterior a nuestra instancia de Nextcloud, añadiríamos debajo de la línea "Document Root" los siguientes datos:

`ServerName cloud.tudominio.com`

Por último, reiniciamos Apache para aplicar los cambios realizados:

`sudo service apache2 restart`

Ya podremos acceder mediante la IP interna de nuestra máquina a nuestra instancia de Nextcloud, donde introduciremos los datos relativos a la base de datos creada anteriormente, y crearemos nuestra cuenta de administrador.

#### 5. Optimización de la instalación

Con los pasos anteriores, ya estará configurado Nextcloud, sin embargo si nos dirigimos a "Configuraciones de Administracion", veremos unos errores que procederemos a corregir:

##### 5.1 Corrección error límite memoria PHP

Editamos el archivo php.ini, modificando la línea "memory-limit" a 512M

    sudo vi /etc/php/8.3/apache2/php.ini
    sudo service apache2 restart

##### 5.2 Error no tener región de teléfonos asociada

Editamos el archivo config.php de nuestro directorio de Nextcloud:

`sudo vi /var/www/nextcloud/config/config.php`

Y añadimos al final estas dos líneas:

    'default_phone_region' => 'ES',
    'memcache.local' => '\OC\Memcache\APCu',

##### 5.3 Problema de resolución "caldav" y "carddav"

Editaremos nuevamente el archivo .conf relativo a nuestro site:

`vi /etc/apache2/sites-available/000-default.conf`

Dejándolo como se indica a continuación:

    <VirtualHost *:80>
    
         DocumentRoot /var/www/nextcloud
    
        <Directory /var/www/nextcloud/>
            Require all granted
            AllowOverride All
            Options FollowSymLinks MultiViews
    
            <IfModule mod_dav.c>
                Dav off
            </IfModule>
    
            RewriteEngine On
            RewriteRule ^/\.well-known/carddav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
            RewriteRule ^/\.well-known/caldav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
            RewriteRule ^/\.well-known/host-meta https://%{SERVER_NAME}/public.php?service=host-meta [QSA,L]
            RewriteRule ^/\.well-known/host-meta\.json https://%{SERVER_NAME}/public.php?service=host-meta-json [QSA,L]
            RewriteRule ^/\.well-known/webfinger https://%{SERVER_NAME}/public.php?service=webfinger [QSA,L]
    
        </Directory>
    
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    </VirtualHost>

Tras esto, habilitarémos el módulo añadido y reiniciaremos Apache:

    sudo a2enmod rewrite
    service apache2 restart

##### 5.4 Incidencia módulo PHP-IMAGICK

Este módulo fue instalado durante la preparación de la VM, pero no es reconocido por la instalación de la plataforma. Lo eliminamos e instalamos nuevamente:

    sudo apt remove imagemagick-6-common
    sudo apt remove php-imagick
    sudo apt autoremove
    sudo apt install php-imagick imagemagick
    service apache2 restart

##### 5.5 Configuración SSL y navegación segura

Editaremos el archivo default-ssl.conf . Si disponemos de un dominio y un certificado SSL válido, editarlo de forma correspondiente para que apunte a su ubicación.

`vi /etc/apache2/sites-available/default-ssl.conf`

Y dejaremos el archivo como se muestra a continuación:

    <IfModule mod_ssl.c>
            <VirtualHost *:443>
                    ServerName nextcloud.tudominio.com
                    SSLEngine On
                    SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
                    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
                    DocumentRoot /var/www/nextcloud/
    
            <Directory /var/www/nextcloud/>
                    Options +FollowSymlinks
                    AllowOverride All
    
            <IfModule mod_dav.c>
                    Dav off
             </IfModule>
    
                    SetEnv HOME /var/www/nextcloud
                    SetEnv HTTP_HOME /var/www/nextcloud
                    RewriteEngine On
                    RewriteRule ^/\.well-known/carddav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
                    RewriteRule ^/\.well-known/caldav https://%{SERVER_NAME}/remote.php/dav/ [R=301,L]
                    RewriteRule ^/\.well-known/host-meta https://%{SERVER_NAME}/public.php?service=host-meta [QSA,L]
                    RewriteRule ^/\.well-known/host-meta\.json https://%{SERVER_NAME}/public.php?service=host-meta-json [QSA,L]
                    RewriteRule ^/\.well-known/webfinger https://%{SERVER_NAME}/public.php?service=webfinger [QSA,L]
    
            </Directory>
    
            <IfModule mod_headers.c>
                    Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
            </IfModule>
    
                    ErrorLog ${APACHE_LOG_DIR}/error.log
                    CustomLog ${APACHE_LOG_DIR}/access.log combined
    
            </VirtualHost>

Una vez salvado, cargamos los módulos correspondientes y reiniciamos Apache:

    sudo a2ensite default-ssl.conf
    sudo a2enmod ssl
    sudo a2enmod rewrite
    service apache2 restart

##### 5.6 Configuración CRON

Si nos dirigimos a "Ajustes Básicos" dentro de nuestra instancia de Nextcloud, veremos que aparece AJAX habilitado, y nos recomienda cambiarlo a CRON. Lo cambiamos, y configuraremos el cronjob:

`sudo crontab -u www-data -e`

Seleccionamos la opción 1 (/bin/nano), y añadimos esta línea al final del archivo:

`*/5  *  *  *  * php -f /var/www/nextcloud/cron.php`

Con esto habremos finalizado la instalación y configuración de Nextcloud.
