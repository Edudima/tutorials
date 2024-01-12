# Instalación de Heimdall en Ubuntu 22.04
A continuación os muestro una guía paso a paso para instalar Heimdall en **Ubuntu Server 22.04**. 

Generalmente el deploy de este tipo de aplicaciones se realiza en contenedores, pero aquí os enseño esta alternativa.

Heimdall es un dashboard para mantener organizadas todas nuestras aplicaciones y accesos web.

Además se encuentra en continua actualización y mantenimiento, agregando nuevas funcionalidades en forma de "enhanced apps". 

##### 1.  Descarga Heimdall

    RELEASE=$(curl -sX GET "https://api.github.com/repos/linuxserver/Heimdall/releases/latest" | awk '/tag_name/{print $4;exit}' FS='[""]'); echo $RELEASE &&\
    curl --silent -o ${RELEASE}.tar.gz -L "https://github.com/linuxserver/Heimdall/archive/${RELEASE}.tar.gz"

##### 2.  Comprueba la versión descargada

    ls *.tar.gz

##### 3.  Descomprime el archivo descargado

    tar xvzf V2.5.8.tar.gz

:warning: En este caso, la versión descargada es la V2.5.8 .Ajusta el comando anterior con la versión que hayas descargado. 

##### 4.  Instala las dependencias PHP de Heimdall

    sudo apt install php-sqlite3 php-zip
    apt-get install php

##### 5.  Crea heimdall.service

    vi /etc/systemd/system/heimdall.service

y copia el siguiente texto con los ajustes pertinentes en *User, Group, WorkingDirectory y Port*

    [Unit]
    Description=Heimdall
    After=network.target
    [Service]
    Restart=always
    RestartSec=5
    Type=simple
    User=tuusuario
    Group=tugrupo
    WorkingDirectory=eldirectoriodeheimdall
    ExecStart="/usr/bin/php" artisan serve --host 0.0.0.0 --port tupuerto
    TimeoutStopSec=30
    [Install]
    WantedBy=multi-user.target

##### 6. Habilita el servicio

    systemctl enable heimdall.service
    systemctl daemon-reload

Ya solo queda acceder vía web mediante *http://tuip:tupuerto* y comiences a añadir aplicaciones a tu dashboard!
