---
title: How to easily install HASS.core on Raspberry Pi 4 running raspbian
tags: [Raspbian, Raspberry Pi 4, Home-assistant, HASS.core, HASS, Installation script, guide, raspbian buster, latest raspbian, newbie, beginner]
style: border
color: primary
description: How to easily install HASS.core using bash scripts on Raspberry Pi 4 running raspbian.
---
Source: [KRDesigns.com](https://krdesigns.com)

## Why this guide
I wrote this article to help people who would like to install HASS.core (home-assistant) without having to understant Unix system. I will try my best to give the step-by-step installation, so the beginner will be able to have HASS running in no time.

I understand there are many guide out there. What makes mine different is because I will include a bash script which will make the installation a brezze.

## The setup
First you will need to download the official raspbian buster image from [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/). Since I just wanna run it without windows then I choose Raspbian Buster Lite and run it headless. Next is to burn the image to your microSD using [Balena etcher](https://www.balena.io/etcher/). 

Follow etcher step to burn microSD and ince completed your microSD will be ejected. So please remount your microSD and you will need to add `ssh` into the root directory on microSD (microSD) The `ssh` files will allow you to install raspberry headless (installation via SSH)

Now lets begin to insert your microSD and boot your RPI, furthermore plugin your network cable to your RPI to be sure the installation when smoothly. Please wait until you see your network light begin to blinking meaning that your RPI is now up and running. 

Now you are ready to connect remotely to your RPI via SSH. Now you will have to connect to your Raspberry from your PC/Mac/Linux. Windows user can used putty to replace terminal. You can connect to your RPI IP using this `ssh pi@192.168.1.x` and its default password `raspberry`

Once connected to your RPI you will need to create a root password by typing `sudo passwd` please type the new password twice (the screen will not show your typing but it will record your password) After the root password created you can change to user root by typing `su` and input your password. Alright you are now running RPI as root and be sure to type `~` so you can go to root home.

Now its time for you to download my scripts and begin the installation and setting up and securing your RPI. Simply run `curl -sL https://raw.githubusercontent.com/krdesigns-com/hass-install/master/install.sh | bash` 
and the installation will started until your system rebooted.

Now you need to login back to your RPI using SSH to your RPI IP `ssh root@192.168.1.x` this time you will be asked for your public key password. Warning pi user is now unable to login since we secure our systems. 

## Installing HASS.core
Now that we are set this time you will need to run the 2nd scripts to install Home-Assistant core by typing `./home-assistant.sh` It will run and install the rest of installation. Upon its compleation you can connect to your home-assistant via http://ip:8123 should it fail you should connect to portainer to check what when wrong via http://ip:9000 

## More AddOns is available
I prepare extra script to allow you to install more addons. The scripts is located inside hass directory. To install simply `cd hass/` and you can see the list by typing `ls -la` To run the script your can type `./thenameofaddone.sh` for example `./watchtower-install.sh`

Okay.. I hope this short guide will be able to help you out. Please let me know on disquss if you need my help or via github issue.