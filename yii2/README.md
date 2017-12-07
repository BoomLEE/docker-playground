## Goal
- Yii2 + PHP5.6 + MySQL5.6 + Apache

## Docker Images
Docker Hub                      | Container Tag
--------------------------------|-----------------------------|
https://hub.docker.com/_/php/   | php:5.6-apache
https://hub.docker.com/_/mysql/ | mysql:5.6

## Directory
```
project
├── Dockerfile
│     config
│     └── apache2
│         └── sites-available
│             ├── 000-default.conf
│             └── default-ssl.conf
├── docker-compose.yml
└── yii-app
     └── yii
```

## Setting

### httpd.conf
```
% mkdir -p config/apache2/sites-available
% cd config/apache2/sites-available
% docker run --name php-apache-tmp php:5.6-apache true
% docker cp php-apache-tmp:/etc/apache2/sites-available/000-default.conf .
% vi 000-default.conf
差分は以下。
-  DocumentRoot /var/www/html
+  DocumentRoot /var/www/html/public

% docker cp php-apache-tmp:/etc/apache2/sites-available/default-ssl.conf .
% vi default-ssl.conf
差分は以下。
-    DocumentRoot /var/www/html
+    DocumentRoot /var/www/html/public

% docker rm php-apache-tmp
```
### Dockerfile
```
FROM php:5.6-apache

# Install PHP
RUN apt-get update \
    && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        openssl libssl-dev \
        libxml2-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt pdo_mysql mbstring xml tokenizer zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# turn on apache mod_rewrite
RUN cd /etc/apache2/mods-enabled \
    && ln -s ../mods-available/rewrite.load

# install composer
RUN cd /usr/bin && curl -s http://getcomposer.org/installer | php && ln -s /usr/bin/composer.phar /usr/bin/composer
RUN apt-get install -y git && composer global require "yii2/installer=~1.1"

WORKDIR /var/www/html
```
### docker-compose.yml
```
yii2:
  container_name: yii2
  build: .
  ports:
   - "80:80"
  environment:
    PATH: ${PATH}:/usr/local/bin:/root/.composer/vendor/bin
  volumes:
    - ./yii2-app:/var/www/html
    - ./config/apache2/sites-available:/etc/apache2/sites-available
  links:
    - yii2-db

yii2-db:
  image: mysql:5.6
  container_name: yii2-mysql5.6
  command: mysqld --character-set-server=utf8 --collation-server=utf8_general_ci --init-connect="SET NAMES utf8" --innodb_file_per_table=1 --innodb_file_format=BARRACUDA
  environment:
    MYSQL_ROOT_PASSWORD: rootpassword
    MYSQL_DATABASE: yii2
    MYSQL_USER: local
    MYSQL_PASSWORD: password
  ports:
    - "13306:3306"
  volumes_from:
    - yii2-dbdata

yii2-dbdata:
  image: mysql:5.6
  container_name: yii2-mysql5.6-dbdata
  command: echo "Data-only container for yii2 MySQL5.6"
```
