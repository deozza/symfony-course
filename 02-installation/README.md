# Installation

## Sommaire

<!--toc:start-->
- [Installation](#installation)
  - [Sommaire](#sommaire)
  - [Prérequis](#prérequis)
  - [Fichiers de configuration](#fichiers-de-configuration)
  - [Vérification de l'installation de l'environnement de développement](#vérification-de-linstallation-de-lenvironnement-de-développement)
  - [Création d'une application](#création-dune-application)
    - [Via la CLI Symfony](#via-la-cli-symfony)
    - [Via composer](#via-composer)
    - [Installation des dépendances](#installation-des-dépendances)
    - [Vérification de l'installation de l'application](#vérification-de-linstallation-de-lapplication)
  - [Sources](#sources)
<!--toc:end-->

## Prérequis

- avoir installé docker engine
- avoir installé l'extension docker compose
- pour macos :
  - avoir installé Docker Desktop
- pour windows :
  - avoir installé et configuré WSL Ubuntu
  - tout le développement devra se faire depuis le WSL, pas sur l'invite de commande ni le powershell

Organiser ses fichiers de la manière suivante :

```bash
├── app 
├── docker-compose.yml 
├── Dockerfile
├── logs
└── mysql
```

## Fichiers de configuration

Fichier Dockerfile :

```Dockerfile
FROM php:8.5-apache

# PHP
RUN apt-get update -y && apt-get upgrade -y
RUN apt-get install -y zlib1g-dev libwebp-dev libpng-dev && docker-php-ext-install gd
RUN apt-get install libzip-dev -y && docker-php-ext-install zip
RUN docker-php-ext-install pdo pdo_mysql mysqli

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Symfony CLI
RUN curl -sS https://get.symfony.com/cli/installer | bash

# Nodejs
RUN curl -fsSL https://deb.nodesource.com/setup_22.x -o nodesource_setup.sh
RUN bash nodesource_setup.sh
RUN apt-get install -y nodejs

# Apache
RUN a2enmod rewrite
RUN service apache2 restart

EXPOSE 80
```

Fichier docker-compose.yml :

```yml
services:
     app:
          container_name: app
          build:
               context: .
               dockerfile: Dockerfile
          volumes: 
               - ./app:/var/www/html
               - ./logs:/var/log/apache2/
          ports:
               - 8080:80
               - 8000:8000
     mysql:
          image: mysql:latest
          container_name: mysql
          restart: unless-stopped
          environment:
            MYSQL_DATABASE: 'db'
            # So you don't have to use root, but you can if you like
            MYSQL_USER: 'user'
            # You can use whatever password you like
            MYSQL_PASSWORD: 'password'
            # Password for root access
            MYSQL_ROOT_PASSWORD: 'password'
          ports:
            - '3306:3306'
          volumes:
            - ./mysql:/var/lib/mysql
     

```

Toutes les commandes seront lancées à partir du conteneur docker

## Vérification de l'installation de l'environnement de développement

```bash
docker compose up -d --build
docker compose exec app php -v
docker compose exec app composer -v
docker compose exec app symfony check:requirements
```

## Création d'une application

### Via la CLI Symfony

```bash
docker compose exec app symfony new ./ --version="8.1.*"
```

### Via composer

```bash
docker compose exec app composer create-project symfony/skeleton:"8.1.*" ./
```

### Installation des dépendances

```bash
docker compose exec app composer install
```

### Vérification de l'installation de l'application

```bash
docker compose exec app php bin/console about
```

## Sources

- https://symfony.com/doc/current/setup.html#technical-requirements
- https://symfonycasts.com/screencast/symfony/setup
