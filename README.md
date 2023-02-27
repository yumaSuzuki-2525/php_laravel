# php_laravel
laravel学習用


# 環境構築

https://nochio12.hatenablog.com/entry/2020/10/02/013014

```cmd
project
┣ docker
┃ ┣ db
┃ ┣ nginx
┃ ┗ php
┣ server
┗docker-compose.yml
  ```

上記のディレクトリを作成。

```cmd
mkdir docker
mkdir docker/db
mkdir docker/nginx
mkdir docker/php
touch docker/php/php.ini
mkdir server
touch docker-compose.yml
touch docker/nginx/default.conf
touch docker/php/Dockerfile
```

## docker-compose.yml

``` yml
version: '3'

services:
  php:
    container_name: php
    build: ./docker/php
    volumes:
    - ./server:/var/www/html

  nginx:
    image: nginx
    container_name: nginx
    ports:
    - 50080:80
    volumes:
    - ./server:/var/www/html
    - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
    - php

  db:
    image: mysql:5.7
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: database
      MYSQL_USER: docker
      MYSQL_PASSWORD: docker
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
    - ./docker/db/data:/var/lib/mysql
    - ./docker/db/my.cnf:/etc/mysql/conf.d/my.cnf
    - ./docker/db/sql:/docker-entrypoint-initdb.d
    ports:
    - 53306:3306
```

## /nginx/default.conf

```conf
server {
  listen 80;
    index index.php index.html;
    root /var/www/html/public;

  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }

  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass php:9000;
    fastcgi_index index.php;
    include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root/index.php;
      fastcgi_param PATH_INFO $fastcgi_path_info;
  }
 }
 ```

## php/Dockerfile

 ```Dockerfile
FROM php:7.4-fpm
COPY php.ini /usr/local/etc/php/


#Composer install
RUN cd /usr/bin && curl -s http://getcomposer.org/installer | php && ln -s /usr/bin/composer.phar /usr/bin/composer

RUN apt-get update \
&& apt-get install -y \
git \
zip \
unzip \
vim
RUN apt-get update \
    && docker-php-ext-install pdo_mysql

ENV COMPOSER_ALLOW_SUPERUSER 1

ENV COMPOSER_HOME /composer

ENV PATH $PATH:/composer/vendor/bin


WORKDIR /var/www

RUN composer global require "laravel/installer"
```

## php/php.ini

```ini
[Date]
date.timezone = "Asia/Tokyo"
[mbstring]
mbstring.internal_encoding = "UTF-8"
mbstring.language = "Japanese"
```


`docker compse up -d`  
にて立ち上げ

`docker compose exec php bash`  
にてコンテナに入り

cd html
`composer create-project --prefer-dist laravel/laravel .`
にてLaravelインストール

 chmod -R 777 storage
 にて権限付与

 php artisan serve  
 にて起動

デフォルトポートを変更しているので、  
 http://localhost:50080  
 にアクセス


 ```

docker exec -it laravel_db  /bin/bash
mysql -u root -p
(パスワードはroot)
create database laravel;

```


ctr + p -> .env にて.envを開き
編集
```php
DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
DB_HOST=db
DB_PORT=53306
DB_DATABASE=laravel # さっき作ったDB名
DB_USERNAME=root
DB_PASSWORD=root
```

```
docker compose exec php bash

