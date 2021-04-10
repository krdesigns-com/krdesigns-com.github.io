---
title: Install and run Monitor Docker plugin on Home-Assistant Supervised version the right way.
tags: [home-assistant, HA, Debian, Supervised, Monitor Docker, Plugin, HACS]
style: border
color: primary
description: Install and run Monitor Docker plugin on Home-Assistant Supervised version the right way.
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Home-Assistant](https://home-assistant.io), [Monitor Docker](https://github.com/ualex73/monitor_docker), [HACS](https://hacs.xyz/), [Portainer](https://hub.docker.com/r/portainer/portainer-ce), [Sami Jawhar project](https://github.com/sjawhar/docker-socket-proxy)


## SOFTWARE
- [Home Assistant](https://www.home-assistant.io)
- [HA Supervised](https://github.com/home-assistant/supervised-installer)
- [HACS](https://hacs.xyz/)
- [Monitor Docker](https://github.com/ualex73/monitor_docker)
- [Portainer](https://hub.docker.com/r/portainer/portainer-ce)

## Why this guide?
Monitor Docker is a very nice plugin if you would like to add plugins/sensor to monitor all your docker running on your network. So if you are running **Docker Home-Assistant core** you will be able to run it. However like my self and many other, we want to have more feature for our Home-Assistant which the team called it Supervised version. Supervised version run with strict rules which exclude Home-Assistant running with `/var/run/docker.sock` in which required by **Monitor Docker** to get the sensor information. So, up until last week I have not find a better way to install it without having to modified HA Docker volume detail everytime HA being updated to a new version.

## How do I do it:
I fork [Sami Jawhar project](https://github.com/sjawhar/docker-socket-proxy) build more support for Raspberry Pi 32bit and 64bit, have the container available on Docker Hub, this way I can help anyone from Home-Assistant community to enjoy **Monitor Docker** without any problem.

1. You will need to install your [HA Supervised](https://github.com/home-assistant/supervised-installer) version. If you need help on installing this version for Raspberry Pi 4, you can follow my guide [here](https://krdesigns.com/articles/installation-home-assistant-with-supervisor-on-debian-10)

2. Install [HACS](https://hacs.xyz/) and [Monitor Docker](https://github.com/ualex73/monitor_docker)

3. For a better docker control I will also install [Portainer](https://hub.docker.com/r/portainer/portainer-ce) on my server.

4. create a local TLS certificate following the docker [guide](https://docs.docker.com/engine/security/protect-access/) and make sure to copy the certificate into your server or better create one inside you machine.

5. install ranrinc [docker-socker-proxy](https://github.com/ranrinc/docker-socket-proxy) by going to your HA server CLI and type `docker run --name docker-socket-proxy --restart always -d -p 2376:2376 -v /var/run/docker.sock:/var/run/docker.sock -v /<YOUR-DOCKER>/docker/certs:/run/secrets ranrinc/docker-socket-proxy`

6. Make a new directory `.certs` inside your HA config directory and make sure HA able to read that directory. Copy 3 files into your `.certs` directory from your `/<YOUR-DOCKER>/docker/certs` The 3 files are `ca.pem`, `cert.pem`, and `key.pem`.

7. Add Monitor Docker config with this setup
```yaml
monitor_docker:
  - name: Docker
    url: tcp://<YOUR-IP>:2376
    certpath: '/config/.certs'
    # containers:
    rename:
      plex: Plex
    # monitored_conditions:
```

8. Reboot your **Home-Assistant** and if everything working correctly you should be able to get individual sensor appearing on your Home-Assistant states.

9. Thats its! Oh the best part for **docker-socker-proxy** it can run on a multiple Docker server, this way you could also monitor those containers too. For those machine you dont need to create a new certificate since you can used the same one that you just create for the first docker. IF you used the same one, you dont need a multiple certs directory for each server. 

P.S I include **Monitor Docker** config for you to follow:
```yaml
monitor_docker:
  - name: DockerOne
    url: tcp://<YOUR-1stIP>:2376
    certpath: '/config/.certs'
    # containers:
    rename:
      plex: Plex
    # monitored_conditions:
  - name: DockerTwo
    url: tcp://<YOUR-2ndIP>:2376
    certpath: '/config/.certs'
    # containers:
    rename:
      plex: Plex
    # monitored_conditions:    
```

## The complete Monitor Docker in actions 
![monitordocker](https://community-assets.home-assistant.io/original/3X/b/5/b58a032b656051bb45e773dff399a973e1393ab3.png "Monitor Docker")

## Closing
I hope this guide will help you install Monitor Docker into your Home-Assistant Supervised version and you will be always able to run it without having to modified HA Docker container setup manually. 
