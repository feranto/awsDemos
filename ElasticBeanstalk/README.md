# Using AWS Elastic Beanstalk to deploy a NodeJS API

In this tutorial we will show you how to leverage AWS Elastic Beanstalk to deploy and run a NodeJS/Express/SwaggerUI API.

## Basic requirements ##

*   Git [git](https://git-scm.com/download/win)
*   A bash terminal (could be Git Bash if on Windows)
*  [Visual Studio Code](https://code.visualstudio.com/download)
*  [NodeJS](https://nodejs.org/en/download/)
*  [Python](https://www.python.org/downloads/)

## Donwloading and Running the NodeJS API Locally ##

1.  The NodeJS/Express/SwaggerUI api code we will be deploying can be located in this [repo](https://github.com/feranto/NodeJSExpressSwaggerUIBackend). (All the details of how to recreate this API and file structure can be located in the repo README).


![alt text][repo]

[repo]: images/1.png  "Recursos en vscode"


2.  We clone the repo locally ``` git clone https://github.com/feranto/NodeJSExpressSwaggerUIBackend.git ```
3.  We access the repo folder ``` cd NodeJSExpressSwaggerUIBackend```
3.  We run ``` npm install ``` to download the node modules
4.  We run ``` node server.js ``` to start the API in a local server
5.  Now we can access the api locally by opening the [localhost url](http://localhost:8000) ``` http://localhost:8000 ```
6.  Good, now that we have our API running locally, now we prepare to deploy it to AWS Elastic Beanstalk.

## Installing and configuring AWS-cli ##
1.  First we have to install AWS-Cli with the [following steps](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) 
2.  After installing successfully AWS-Cli we must configure an access-key to be able to manager our AWS resources via AWS Cli. You can use the [following steps](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey).


## ##
Configuramos EB-cli http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html
    


``` docker info ``` - Nos la información de nuestro entorno

``` docker pull ``` - Trae una imagen del registro

``` docker images ```- Muestra las imagenes Docker disponibles

``` docker ps ```- Lista los contenedores Docker que están corriendo

``` docker build ```- Construye una imagen Docker

``` docker run ```- Utiliza una imagen para correr un contenedor

``` docker exec ```- Ejecuta un comando en el contenedor

``` docker stop ```- Detiene un contenedor que está corriendo


## Definiendo nuestro contenedor con DockerFile ##

1.  Lo primero que debemos hacer es abrir una terminal(ya sea CMD o Powershell) y asegurarnos que nuestro entorno docker esté funcionando bien, para ello corremos el comando ``` docker info ``` y nos debería mostrar algo como lo siguiente:

![alt text][dockerinfo]

[dockerinfo]: imagenes/dockerInfo.PNG  "Comando Docker Info"

2.  Luego debemos bajar los scripts, archivos y configuraciones para nuestro contenedor y para ello ingresamos al link de este [repositorio](https://github.com/feranto/tutoriales-docker) haciendo click en en el siguiente botón:

![alt text][gitcode]

[gitcode]: imagenes/descargaCodigo.png  "Código Github"


3. Luego descomprimimos la carpeta y abrimos la carpeta con Visual Studio Code y deberíamos ver lo siguiente:

![alt text][vscode]

[vscode]: imagenes/vscode1.PNG  "Recursos en vscode"


4. Ahora veremos acerca de Dockerfile. Te explicamos que significa cada comando dentro del Dockerfile:

```FROM``` - Especifica la imagen base que el Dockerfile utiliza para construir la nueva imagen. Para est ejemplo estamos usando la imagen phusion/baseimage como base ya que es un Ubuntu minimalista modificado para ser amigable con Docker.

```MAINTAINER``` - Especifica el nombre del autor y su email.

```RUN``` - Corre un cualquier comando UNIX necesario para construir la imagen.

```ENV``` - Define las variables de entorno. Para este ejemplo usamos algunas como JAVA_HOME y la seteamos a los valores que necesitamos.

```CMD``` - Provee la facilidad de correr comandos al inicio del contenedor. Puede ser sobrescrito con el comando run de docker.

```ADD``` - Esta instrucción copia los archivos nuevos, directorios al sistema de archivos del contenedor Docker en las direcciones especificadas.

```EXPOSE``` - Esta instrucción expone puertos del contenedor a la maquina host.


Ahora abrimos el Dockerfile que debería tener el siguiente contenido:

```c
# Dockerfile

FROM  phusion/baseimage:0.9.17

MAINTAINER  feranto <fernando.mejia@microsoft.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu trusty main universe" > /etc/apt/sources.list

RUN apt-get -y update

RUN DEBIAN_FRONTEND=nonisnteractive apt-get install -y -q python-software-properties software-properties-common

ENV JAVA_VER 8
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

RUN echo 'deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main' >> /etc/apt/sources.list && \
    echo 'deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main' >> /etc/apt/sources.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C2518248EEA14886 && \
    apt-get update && \
    echo oracle-java${JAVA_VER}-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections && \
    apt-get install -y --force-yes --no-install-recommends oracle-java${JAVA_VER}-installer oracle-java${JAVA_VER}-set-default && \
    apt-get clean && \
    rm -rf /var/cache/oracle-jdk${JAVA_VER}-installer

RUN update-java-alternatives -s java-8-oracle

RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-oracle" >> ~/.bashrc

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


##maven installation
ENV MAVEN_VERSION 3.3.9

RUN mkdir -p /usr/share/maven \
  && curl -fsSL http://apache.osuosl.org/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz \
    | tar -xzC /usr/share/maven --strip-components=1 \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven

VOLUME /root/.m2

##tomcat installation
RUN apt-get update && \
    apt-get install -yq --no-install-recommends wget pwgen ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
ENV TOMCAT_MAJOR_VERSION 8
ENV TOMCAT_MINOR_VERSION 8.0.11
ENV CATALINA_HOME /tomcat
RUN wget -q https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz && \
	wget -qO- https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz.md5 | md5sum -c - && \
	tar zxf apache-tomcat-*.tar.gz && \
 	rm apache-tomcat-*.tar.gz && \
 	mv apache-tomcat* tomcat

##application and scripts installation
ADD create_tomcat_admin_user.sh /create_tomcat_admin_user.sh
ADD springwebapp.war /tomcat/webapps/springwebapp.war
RUN mkdir /etc/service/tomcat
ADD run.sh /etc/service/tomcat/run
RUN chmod +x /*.sh
RUN chmod +x /etc/service/tomcat/run

EXPOSE 8080

CMD ["/sbin/my_init"]

```

## Creamos nuestra imagen ##
1.  Una vez que hemos definido nuestro contenedor en un Dockerfile ahora procedemos a crear una imagen a partir de el con el siguiente comando:
``` cd recursos```
``` docker build -f Dockerfile -t ejemploferanto/ubuntujavamaventomcatspringmvc:1 .```

Estos nos correrá un proceso en el cual ejecutará todos nuestro comandos, copiará todos nuestros archivos y configurará nuestras variables. Como resultado final, en nuestra maquina local generará la imagen  ```ejemploferanto/ubuntujavamaventomcatspringmvc:1``` en nuestro sistema.

![alt text][imagebuild1]

[imagebuild1]: imagenes/imageBuild.png  "Construcción imagen"

![alt text][imagebuild2]

[imagebuild2]: imagenes/imageBuild2.png  "Construcción imagen"

Ahora deberíamos poder ejecutar el siguiente comando y ver nuestra imagen alli:

``` docker images```

![alt text][imagebuild3]

[imagebuild3]: imagenes/imageBuild3.png  "Construcción imagen"

## Corremos nuestro contenedor ##

1.  Finalmente una vez tenemos nuestra imagen creada procedemos a correrla con el siguiente comando:

``` docker run -it --name miappjava -p 8080:8080 ejemploferanto/ubuntujavamaventomcatspringmvc:1 bash -c "/tomcat/bin/catalina.sh run" ```

![alt text][imagerunning1]

[imagerunning1]: imagenes/dockerCorriendo1.png  "Imagen Corriendo"

También podemos correrlo en modo "detached" es decir podemos cerrar la consola y seguirá funcionando con el parametro ``` -d ```.

2.  Podemos ver nuestro contenedor corriendo localmente utilizando el comando ``` docker ps ```

3.  Finalmente accedemos nuestra app con la siguiente URL:
[http://localhost:8080/springwebapp/car/add](http://localhost:8080/springwebapp/car/add)

![alt text][imagerunning2]

[imagerunning2]: imagenes/dockerCorriendo2.png  "Imagen Corriendo"


## Publica nuestra imagen en un registro(Dockerhub) ##

1.  Ahora la imagen creada la publicaremos en un registro para que otros puedan correrla. Para esto es necesario crear una cuenta en [DockerHub](https://hub.docker.com/)

2.  Luego Hacemos login con nuestra cuenta de docker hub:

```docker login --username=yourhubusername ```

2.  Luego procedemos a obtener nuestras imagenes locales:
``` docker images ```

3.  Luego agregamos etiquetas a nuestra imagen
``` docker tag bb38976d03cf yourhubusername/verse_gapminder:firsttry ```

4.  Luego hacemos push a nuestra imagen
``` docker push yourhubusername/verse_gapminder ```

5.  Finalmente deberá aparecer nuestra imagen en Dockerhub:
Imagen en dockerhub: https://hub.docker.com/r/feranto/java-maven-spring/