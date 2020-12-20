[Retour à la racine du projet](https://github.com/EPradillon/veille-informatique)   

[Retour au dossier Dev-Ops](https://github.com/EPradillon/veille-informatique/tree/main/dev-ops)  

https://github.com/EPradillon/veille-informatique/tree/main/dev-ops
# Step by step : ('depuis un système ubuntu')

## 1. Choix pour le déploiement
Docker permetra de reproduire l'infrastructure si besoin.
Technologie parfaite pour le scaling.
>De plus on pourra faire des extensions sans interruption de service

### 1.1 Installation de docker et docker compose
$apt-get install docker docker-compose

### 1.2 structure des dossiers : 
  - apps/
    - my-symfony-app/
  - bin/ 
  - docker/ 
  - .env 
  - docker-compose.yml

## 2. Contenu des différents dossier et fichier :

###  2.1 Elements du docker compose :
```yml
version:  '3.7'

# database pour le portail (mysql par exemple)
services:
mysql:
    image: mysql:8.0 #https://hub.docker.com/
    restart: on-failure
    environment:
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: example
    version: '3.7'

# Si on a besoin d'une interface pour la database    
services:
  mysql: 
    …

  adminer:
    image: adminer #https://hub.docker.com/
    restart: on-failure
    ports:
    - '8080:8080'

# http serveur :
version: '3.7'
services:
  mysql: 
    …
# Image spécifique : Alpine (version légère de linux)
  nginx:
    image: nginx:1.19.0-alpine #https://hub.docker.com/
    restart: on-failure
    # On va pouvoir partager un volume commun entre le container et notre système
    # Si l'environnement a besoin de changer régulièrement on peut ainsi editer le './apps/my-symfony-app/public/'
    # Et le docker "subira" les changements sans avoir à redémarer
    volumes:
      # Volume : (cf  structure des dossiers + pour symfo 4 on partage le dossier "public")
      - './apps/my-symfony-app/public/:/usr/src/app'
      # fichier de configuration de nginx pour symfony
      - './docker/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro'
      # link des ports qu'on va choisir d'associer (exemple avec Localhost)
    ports:
      - '80:80'
    depends_on: # le container nginx sera construit après celui de php
     - php

  php:
      build:
        context: .
        dockerfile: docker/php/Dockerfile
      volumes: 
        - './apps/my-symfony-app/public/:/usr/src/app'
      restart: on-failure
      env_file:
        - .env
      # pour check son user uid : $ id  (ici on utilisera le uid 1000 et gid 1000)
      user: 1000:1000 
```


### 2.2 ./docker/nginx/default.conf :
```
server {
 server_name ~.*;

 location / {
     root /usr/src/app;

     try_files $uri /index.php$is_args$args;
 }

 location ~ ^/index\.php(/|$) {
     client_max_body_size 50m;

     fastcgi_pass php:9000;
     fastcgi_buffers 16 16k;
     fastcgi_buffer_size 32k;
     include fastcgi_params;
     fastcgi_param SCRIPT_FILENAME /usr/src/app/public/index.php;
 }

 error_log /dev/stderr debug;
 access_log /dev/stdout;
}
```
> source : https://symfony.com/doc/current/setup/web_server_configuration.html




### 2.3 ./docker/php/Dockerfile
```yml
FROM php:7.4-fpm #Php 7.4, parcequ'on aime typer nos variables !

RUN docker-php-ext-install pdo_mysql # librairie PDO de mysql !

RUN pecl install apcu

RUN apt-get update && \
apt-get install -y \
libzip-dev

RUN docker-php-ext-install zip
RUN docker-php-ext-enable apcu

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "if (hash_file('sha384', 'composer-setup.php') === 'e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php --filename=composer \
    && php -r "unlink('composer-setup.php');" \
    && mv composer /usr/local/bin/composer

WORKDIR /usr/src/app

COPY --chown=1000:1000 apps/my-symfony-app /usr/src/app

RUN PATH=$PATH:/usr/src/apps/vendor/bin:bin
```



# 3. pour executer tout ça :
```
$ service  stop  nginx
$ docker-compose build
$ docker-compose up -d
```
# exemple de commande que nous pourrons utiliser :
```
docker-compose exec php composer install 
docker-compose exec php php bin/console doctrine:schema:create
docker-compose exec php php bin/console doctrine:fixtures:load
docker-compose exec php php bin/console assets:install –symlink public/
```


>il est possible de sortir les variables d'environnement du docker compose

>Dans cette configuration les elements de la base de données peuvent etre définitivement perdu
en meme temps que le container.

