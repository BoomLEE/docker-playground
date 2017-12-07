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
