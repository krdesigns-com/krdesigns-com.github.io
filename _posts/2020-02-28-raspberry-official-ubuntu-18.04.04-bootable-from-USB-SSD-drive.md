---
title: Raspberry Pi 4 running Official uBuntu 18.04.04 bootable guide from USB SSD Drive
tags: [Ubuntu Official, Raspberry Pi 4, USB bootable drive, guide, ubuntu 18.04.04 LTS]
style: border
color: primary
description: Guide to setup Raspberry Pi 4 to boot an official uBuntu 18.04.04 from your USB SSD Driver.
---
Source: [KRDesigns.com blogs](https://www.krdesigns.com)

This is a simple tutorial on how to enable your Raspbian Pi 4 boot from SSD Drive running official release ubuntu 18.04.04.

## Why this guide

While waiting for an official method of booting RPI 4 from USB being done by Raspberry Team, this guide will help you running an official ubuntu 18.04.04 OS in a blazing speed on USB 3.1 from SSD Drive instead from microSD. Furthermore it will make sure your data is more safe and secure since SSD is much better for read/write rather then running it from a  microSD.

## The setup

First you will need to download the official ubuntu 18.04.04 (64bit) image from [ubuntu.com](https://ubuntu.com/download/raspberry-pi). Next is to burn the image to your microSD and your SSD Drive using [Balena etcher](https://www.balena.io/etcher/). 

Follow etcher step to burn both drive together (you can actually burn both drive at the same time). Once completed both drive will be ejected. So please remount both drive and you will need to edit both `nobtcmd.txt`. 

Please make sure you read [James A. Chambers Blogs](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) for a guide to enable USB SSD Drive. If it compatible then you should not have any issue. Since mine running using a non compatible enclosure (Orico) then an extra step need to be done to make sure your SSD Drive can be detected by Raspberry Pi.

The original line on `nobtcmd.txt` is 

`net.ifnames=0 dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline rootwait fixrtc` 

and you need to change it to

`net.ifnames=0 dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=/dev/sda2 rootfstype=ext4 elevator=deadline rootwait fixrtc`

or in my case

`usb-storage.quirks=XXXX:XXXX:u net.ifnames=0 dwc_otg.lpm_enable=0 console=ttyAMA0,115200 console=tty1 root=LABEL=writable rootfstype=ext4 elevator=deadline rootwait fixrtc`

This need to be added in my case to allow ubuntu to load SSD driver correctly `usb-storage.quirks=XXXX:XXXX:u` where `XXXX:XXXX` is the code from your SSD Drive ( Again please read [James A. Chambers Blogs](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) guide to figure this out).

Now you can connect both drive to your raspberry pi 4 and turn it on. If everything when well you will have your raspberry pi 4 running ubuntu 18.04.04 LTS on your SSD Drive. Once loaded ubuntu automatically will resize your SSD Drive partition to 100%.

Just to be sure you can check your bootable drive using `findmnt -n -o SOURCE /` where it should show `/dev/sda2` and you can check `df -h` to see the size of you mounted `/` correctly adjusted.

Thats all and I hope this guide will help many of you until Raspberry team decide to allow RPI4 to boot from USB which make this guide depreciated.


