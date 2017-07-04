##haciendo instalación de netbox sobre docker en instalación de ubuntu 16.04.1 sobre Exsi

instalamos ubuntu instalación estandar:
usuario: test
pass: toor

instalación de dependencias:

apt-get install curl
apt-get install ssh


instalamos docker

sudo curl -sSL https://get.docker.com/ | sh

damos permisos a usuario test

sudo usermod -aG docker test

reiniciamos terminal

instalamos vmware tools

apt-get update
apt-get upgrade
apt-get install open-vm-tools-desktop

instalamos docker-compose

Descargamos la imagen de netbox

git clone -b master https://github.com/digitalocean/netbox-docker.git

entramos al directorio

cd netbox

modificamos el arhchivo Dockerfile

para cambiar la rama de la instalación la que trae por defecto no funciona. Tambien cambiamos la línea ```COPY docker/nginx.conf /etc/netbox-nginx/nginx.conf```si no el servidor nginx no funciona.

Debería quedar un archivo de esta manera:

```
FROM python:2.7-alpine

RUN apk add --no-cache \
      bash \
      build-base \
      ca-certificates \
      cyrus-sasl-dev \
      graphviz \
      jpeg-dev \
      libffi-dev \
      libxml2-dev \
      libxslt-dev \
      openldap-dev \
      openssl-dev \
      postgresql-dev \
      wget \
  && pip install gunicorn==17.5 django-auth-ldap

WORKDIR /opt

ARG BRANCH=v2.0.1
ARG URL=https://github.com/digitalocean/netbox/archive/$BRANCH.tar.gz
RUN wget -q -O - "${URL}" | tar xz \
  && ln -s netbox* netbox

WORKDIR /opt/netbox
RUN pip install -r requirements.txt

RUN ln -s configuration.docker.py netbox/netbox/configuration.py
COPY docker/gunicorn_config.py /opt/netbox/
COPY docker/nginx.conf /etc/netbox-nginx/nginx.conf

COPY docker/docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT [ "/docker-entrypoint.sh" ]
```
Modificamos el archivo docker-compose.yml para que redireccione el puerto al 80 quedando el archivo de esta manera:

```
version: '3'
services:
    netbox:
        build: .
        image: digitalocean/netbox:v2.0-beta2
        depends_on:
        - postgres
        env_file: netbox.env
        volumes:
        - netbox-nginx-config:/etc/netbox-nginx/
        - netbox-static-files:/opt/netbox/netbox/static
    nginx:
        image: nginx:1.11-alpine
        command: nginx -g 'daemon off;' -c /etc/netbox-nginx/nginx.conf
        depends_on:
        - netbox
        ports:
        - 80:80
        volumes:
        - netbox-static-files:/opt/netbox/netbox/static
        - netbox-nginx-config:/etc/netbox-nginx/
    postgres:
        image: postgres:9.6-alpine
        environment:
            POSTGRES_USER: netbox
            POSTGRES_PASSWORD: J5brHrAXFLQSif0K
            POSTGRES_DB: netbox
volumes:
    netbox-static-files:
        driver: local
    netbox-nginx-config:
        driver: local
```
ejecutamos la instalación:

```
docker-compose up -d
```
una vez descargado y creados todos los contenedores lo ejecutaremos de la misma manera:

comando | descripción
---|---
docker-compose kill | Mata todos los procesos 
docker-compose ps | Muestra los contenedores en ejecución
docker-compose build | rehace la instalación del paquete si hemos modificado algún parámetro importante
docker-compose stop | para los contenedores
docker-compose up -d | levanta los contenedores de la carpeta donde estemos.

Netbox

para acceder a netbox podemos ir a la dirección de la máquina por el explorador las credenciales por defecto son:
admin / admin