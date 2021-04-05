---
title: Installation Home-Assistant with Supervised on Rapberry Pi 4 running  Debian
tags: [Debian Official, Home-Assistant, Supervised, Raspberry Pi 4, USB bootable drive, guide, Debian 10, Buster]
style: border
color: primary
description: Guide to install Home-Assistant with Supervised on Rapberry Pi 4 running  Debian 10.
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com), [Richard Tirtadji Github](https://github.com/tirtadji-com/rpi_debian_ha_supervised)

## Why this guide

I believe this a very stable method for Raspberry and give you a better control over the docker used rather then just run everything inside your supervised docker. 

I also want to help those nubies with advance installations methods with a step-by-step guide. As a bonus I also add a new scripts which allow you to just run it and it should complete the installation within a minutes.

## The Requirement:
- Rasberry Pi 4
- microSD or SSD Drive (which compatible with linux)
- Debian 10 for RPI [download here](https://raspi.debian.net/tested-images/)
- Raspbian Imager [download here](https://www.raspberrypi.org/software/)
- Your public ssh keys (please google it on how to create one for you)

**Warning!**  
If you are using USB SSD please make sure you read [James A. Chambers Blogs](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) for a guide to enable USB SSD Drive. If it compatible then you should not have any issue. Since mine running using a non compatible enclosure (Orico) then an extra step need to be done to make sure your SSD Drive can be detected by Raspberry Pi.


## The setup

You will need to burn debian images to your microSD/SSD using Raspbian Imager or [Balena etcher](https://www.balena.io/etcher/) on your PC or Mac or Linux. Once complete you will need to detach your microSD/SSD from your computer and attach it back, so you can edit some files inside the `boot` directory.

Once you are on `boot` directory you will need to create a blank ssh files. On macOS or Linux you can simply typing `touch ssh`. You will also need to edit `sysconf.txt` and edit the end of the text files which allow you to login as a `root` on your Debian. This way you can setup the installation headless.

```
# root_pw - Set a password for the root user (by default, it allows
# for a passwordless login)
root_pw=yourpassword

# root_authorized_key - Set an authorized key for a root ssh login.
# input your public ssh key here else you wont be able to login headless
root_authorized_key=your_public_ssh_key
```

**WARNING!**  
Make sure you have your SSH Public Key available else you wont be able to login. Please google to find out how you can create your own SSH public keys.

Now you need to connect your microSD/USB to your Raspberry 4 and start it up (be sure that your network is connected to your Raspberry) Open compelete first run your raspberry network green light should be blinking else it did not get IP or fail to run. Use Router or Network software from your computer to remotely access your Raspberry.

Now SSH yourself into your raspberry from your computer (in my case I run it using mac terminal) by typing `ssh root@the-ip` and then enter your SSH password. 

**WARNING! if you are using SSD as your bootable disk**  
Due to a bugs on Debian 10 Raspberry please make sure you run this command once before you do anything else `apt-mark hold linux-image-arm64`. If you dont then your SSD wont be able to boot, upon reboot.

You are now inside your Raspberry running Debian 10 and now you can begin to update your Debian Package by typing `apt-get update && apt-get dist-upgrade -y && apt autoremove -y` 

Next you will have to install all the requirement for HA + Supervised by typing `apt-get install -y software-properties-common apparmor-utils apt-transport-https ca-certificates curl dbus jq network-manager` then you will need to disable and stop ModemManager by typing `systemctl disable ModemManager` and `systemctl stop ModemManager`

Now you will need to install Docker by typing `curl -fsSL get.docker.com | sh` wait till it finish the installations.

Last but not least to install your HA and supervised by typing 

```
curl -Lo installer.sh https://raw.githubusercontent.com/home-assistant/supervised-installer/master/installer.sh
bash installer.sh --machine raspberrypi4-64
```

Should everything when well you should have HA+Supervised up and running with out any problem by calling it from your browser http://yourip:8123

Thats it, you should now running the latest home-assistant with Supervised on Raspberry on Debian 10.


## Bonus
You want a better installation+security and extra apps+docker then you should try my automation scripts. Go and try it on [github](https://github.com/tirtadji-com/rpi_debian_ha_supervised) Please let me know if you need more help. Enjoy!
