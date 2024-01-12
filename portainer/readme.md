# Instalación de Portainer en Ubuntu 22.04
A continuación os muestro una guía paso a paso para instalar Portainer en **Ubuntu Server 22.04** en pocos minutos.

Con Portainer podrás administrar todos tus contenedores de Docker desde una magnífica interfaz web.

##### 1.  Instala Docker

    sudo apt update
    
    sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    
    sudo apt update
    
    sudo apt-get install docker-ce docker-ce-cli containerd.io

##### 2.  Comprueba que la instalación se haya realizado correctamente

    sudo docker run hello-world

##### 3.  Instala Docker-Compose

    sudo curl -L "https://github.com/docker/compose/releases/download/V2.23.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    
    sudo chmod +x /usr/local/bin/docker-compose

:warning: En este caso, estamos descargando la versión V2.23.3. ¡Puede que esta no sea la versión más reciente cuando leas esto!

##### 4.  Comprueba que Docker-Compose se ha instalado correctamente

    sudo docker-compose --version

##### 5.  Configura Portainer

###### 5.1 Crea un nuevo Docker Volume

    docker volume create datos_portainer

###### 5.2 Arranca el Docker de Portainer

    docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v datos_portainer:/data portainer/portainer-ce
    
