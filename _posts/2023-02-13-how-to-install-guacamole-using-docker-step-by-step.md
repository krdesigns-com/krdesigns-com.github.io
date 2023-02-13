---
title: How to install Guacamole using docker (step-by-step)
tags: [docker, guacamole, linux, ubuntu, debian, docker-compose]
style: border
color: primary
description: Easiest way to install Guacamole (docker) with a detail installation
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Apache Guacamole](https://guacamole.incubator.apache.org/)


## SOFTWARE
- Debian 11 - Bulleye (AMD64 only)
- Latest Docker and Docker Compose

## Why this guide?
Previously there are oznu/docker-guacamole in which allowing docker user to install and run guacamole easily. However that project have been archived and did not get any update since like 3 years ago, so then I'm trying to install them on my own. 

Boy, the installation is quite easy but without doing a lot of google search you will end up with many head pounding and scrateches. The documentation is close to none from Apache Guacamole. This is why I make this guide and hope people can get as much help as they need.

## How do I do it:
##### docker image pull
I will not include on how to install docker and docker-compose because its beyond the scope of this guide. Please also remember since Apache Guacamole did not include other container other then AMD64, ao this guide will not work for ARM (sorry RPI user)

First of all lets download the image from docker-hub, by the time this guide is written the latest tag are 1.4.0 and I will be using MariaDB instead of Postgres.

```
docker pull guacamole/guacamole:1.4.0
docker pull guacamole/guacd:1.4.0
docker pull mariadb:10.9.5
```

##### Grab the latest sql info from the latest images

Run this command to retrive the db.sql

```
docker run --rm guacamole/guacamole:1.4.0 /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

##### Making initial DB docker-compose.yml
using any linux editor - in my case using nano create a new docker-compose.yml. 

```
version: '3'
services:
  guacdb:
    container_name: guacamoledb
    image: mariadb:10.9.5
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'MariaDBRootPass'
      MYSQL_DATABASE: 'guacamole_db'
      MYSQL_USER: 'guacamole_user'
      MYSQL_PASSWORD: 'MariaDBUserPass'
    volumes:
      - './db-data:/var/lib/mysql'
volumes:
  db-data:
```

after saving the files please run `docker-compose up -d`

Next you need to copy the SQL file into the docker container
```
docker cp initdb.sql guacdb:/initdb.sql
```

Last but not least begin to input it to the DB by running this:
```
docker exec -it guacdb bash
cat /initdb.sql | mysql -u root -p guacamole_db
exit
```

and now time for your to turn off the DB by running `docker-compose down`

##### Complete the docker-compose.yml with all the necessary image
Now your best practice is to backup your docker-compose files by typing `cp docker-compose.yml docker-compose.yml.bak` Next you will edit and add more detail

```
version: '3'
services:
  guacdb:
    container_name: guacamoledb
    image: mariadb:10.9.5
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'MariaDBRootPass'
      MYSQL_DATABASE: 'guacamole_db'
      MYSQL_USER: 'guacamole_user'
      MYSQL_PASSWORD: 'MariaDBUserPass'
    volumes:
      - './db-data:/var/lib/mysql'
  guacd:
    container_name: guacd
    image: guacamole/guacd:1.4.0
    restart: unless-stopped
  guacamole:
    container_name: guacamole
    image: guacamole/guacamole:1.4.0
    restart: unless-stopped
    ports:
      - 8080:8080
    environment:
      GUACD_HOSTNAME: "guacd"
      MYSQL_HOSTNAME: "guacdb"
      MYSQL_DATABASE: "guacamole_db"
      MYSQL_USER: "guacamole_user"
      MYSQL_PASSWORD: "MariaDBUserPass"
      TOTP_ENABLED: "true"
    depends_on:
      - guacdb
      - guacd
volumes:
  db-data:
```

Now you will be able to run `docker-compose up -d` and you should have your Guacamole up and running on your server.

## How do I access Guacamole?

Very simple just open your browser and put in your Guacamole IP with port 8080

For example: http://mylocalip.home:8080/guacamole  <- guacamole cant be access via root directory, so you will have to add /guacamole.

The original username/password are `guacadmin/guacadmin`

For security reason I include TOTP in my installation guide, so be sure to have your google authenticator ready for scanning. Else make sure you remove `TOTP_ENABLED: "true"` line from your docker-compose.yml file.

## Closing
I hope this guide will help you install and run Guacamole without any problem. 
