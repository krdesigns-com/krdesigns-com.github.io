---
title: How to install Zeppelin Discord Bot (step-by-step)
tags: [docker, zeppelin, bot, discord, linux, ubuntu, debian, docker-compose]
style: border
color: primary
description: How to install Zeppelin Discord Bot (docker) step-by-step (NGINX + Letsencrypt not on Docker)
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Zeppelin Installation](https://zeppelin.wiki/setup/operating-systems/linux-docker), [Zeppelin Bot](https://github.com/ZeppelinBot/Zeppelin), [Zeppelin Bot](https://zeppelin.gg), [Kewwie Zeppelin Bot Fork](https://github.com/kewwie/zeppelinbot)


## SOFTWARE
- Debian 11 - Bulleye
- Latest Docker and Docker Compose
- NGINX and Certbots
- Fork Zeppelin Bot (workin version)

## Why this guide?
I wanna install Zeppelin Bot on my own server, however somehow the release version is having problem and up-to-date I have not being able to make it run on a new server. Been communicating with the developer and so far zero result.

Furthermore I also want to run my own NGINX + CertBot aside from using Zeppelin nginx package.

## How do I do it:
## Beyond the scope
I will not include on how to install docker, docker-compose, NGINX, and Certbot because its beyond the scope of this guide. However you can follow Zeppelin Bot original guide at [Zeppelin Installation](https://zeppelin.wiki/setup/operating-systems/linux-docker)

## Lets Begin
Be sure you already have Docker, Docker-Compose, NGINX and certbot running.

```
# Add New User and give sudo and docker access
adduser discordbot
usermod -aG sudo discordbot
usermod -aG docker discordbot
```

change to your newly created user
`su - discordbot `

Clone my Zeppelin fork which currently work and I also add the nonginx composer files
```
git clone https://github.com/ranrinc/zeppelinbot
cd zeppelinbot
cp .env.example .env
```
Make your key by
`openssl rand -hex 16`
save the key because you goin to paste it into your `.env` file.

Check your user and group ID
```
id -u
id -g
```
save the userid and groupid because you goin to paste it into your `.env` file.

## Discord Bot Setup
Please refer to [Discord Bot Setup Page.](https://zeppelin.wiki/setup/discord/bot-creation/creation)

## Edit your configurations file (.env)
```
# 32 character encryption key
KEY=xxxxxxxxxxxxxxxxxxxxxxxxx

# Values from the Discord developer portal
CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxx
CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxx
BOT_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxx

# The defaults here automatically work for the development environment.
# For production, change localhost:3300 to your domain.
DASHBOARD_URL=https://sub.yourdomain.com
API_URL=https://sub.yourdomain.com/api

# Comma-separated list of user IDs who should have access to the bot's global commands
STAFF=xxxxxxxxxxxxxxxxxxxxxxxxx

# A comma-separated list of server IDs that should be allowed by default
DEFAULT_ALLOWED_SERVERS=xxxxxxxxxxxxxxxxxxxxxxxxx

# When using the Docker-based development environment, this is only used internally. The API will be available at localhost:DOCKER_DEV_WEB_PORT/api.
API_PORT=3000

# Only required if relevant feature is used
#PHISHERMAN_API_KEY=

# The user ID and group ID that should be used within the Docker containers
# This should match your own user ID and group ID. Run `id -u` and `id -g` to find them.
DOCKER_USER_UID=1001
DOCKER_USER_GID=1001

#
# DOCKER (PRODUCTION)
# NOTE: You only need to fill in these values for running the production environment. See development config above.
#

DOCKER_PROD_DOMAIN=sub.yourdomain.com
DOCKER_PROD_WEB_PORT=443
# The MySQL database running in the container is exposed to the host on this port,
# allowing access with database tools such as DBeaver
DOCKER_PROD_MYSQL_PORT=3001
# Password for the Zeppelin database user
DOCKER_PROD_MYSQL_PASSWORD=xxxxxxxxxxxxxxxxxxxxxxxxx
# Password for the MySQL root user
DOCKER_PROD_MYSQL_ROOT_PASSWORD=xxxxxxxxxxxxxxxxxxxxxxxxx
```
## Run you docker-compose
`docker compose -f docker-compose.nonginx.yml up -d`

if everything work as it should be then your compile docker will stop workin and both your `backend` directory and `dashboard` directory should have `dist` folder. If you did not see them, then you need to recheck your docker logs.


## Setup NGINX conf
```
# I did not add http and https redirect so please add it up yourself
server {
    listen 443 ssl http2;
    server_name sub.yourdomain.com;
    root /home/discordbot/zeppelinbot/dashboard/dist;

    # SSL
    ssl_certificate /etc/letsencrypt/live/sub.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sub.yourdomain.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/sub.yourdomain.com/chain.pem;

    # security
    include custom-snippets/security.conf;

    # logging
    access_log /var/log/nginx/sub.yourdomain.com-access.log;
    error_log /var/log/nginx/sub.yourdomain.com-error.log;

    # index.php
    index index.htm index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location ^~ /api {
        rewrite ^/api(/.*)$ $1 break;
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 600;
        proxy_send_timeout 600;
        proxy_read_timeout 600;
        send_timeout 600;
        client_max_body_size 50M;
    }
}
```

If everything working properly you should be able to reach your dashboard via https://sub.yourdomain.com and also able to run config dashboard.

## Rebuild Docker
Be sure you kill all your docker container
`docker compose -f docker-compose.nonginx.yml down -v`

and then you can rebuild it using
`docker compose -f docker-compose.nonginx.yml up --build -d`

P.S.
In some cases, I will remove all the `zeppelinbot` directory and redo the clone just to be sure the DB datadump get wipeout.

## Closing
I hope this guide will help you install and run Zeppelin Discord Bot easily without any problem. Please ask me any help via comment or [Zeppelin Hangar](https://discord.gg/6UgSqvjy)
