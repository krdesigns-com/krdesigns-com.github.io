---
title: Raspberry Pi 4 running Official Raspbian Buster bootable guide from USB SSD Drive
tags: [Raspbian, Raspberry Pi 4, USB bootable drive, guide, raspbian buster, latest raspbian, beta eeprom]
style: border
color: primary
description: Guide to setup Raspberry Pi 4 to boot an official Raspbian Buster from your USB SSD Driver (currently in beta).
---
Source: [Raspberry Pi Forum](https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=274595)


This is a my second tutorial on how to enable your Raspbian Pi 4 to boot from SSD Drive running the latest raspbian buster (2020-02-13).

### WARNING THIS IS BETA EEPROM DO IT AT YOUR OWN RISK

## Why this guide

Just want to have a fast and secure way of running my RPI 4 instead of needing microSD which much much slower in terms of speed and also in my case is very unreliable to run my home-assistant.

## The setup

First you will need to download the official raspbian buster image from [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/). Since I just wanna run it without windows then I choose Raspbian Buster Lite and run it headless. Next is to burn the image to your microSD and your SSD Drive using [Balena etcher](https://www.balena.io/etcher/). 

Follow etcher step to burn both drive together (you can actually burn both drive at the same time). Once completed both drive will be ejected. So please remount both drive and you will need to add `ssh` into the root directory on both disk (microSD and SSD)

Please make sure you read [James A. Chambers Blogs](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) for a guide to detect your USB SSD Drive. If it compatible then you should not have any issue. Recently I replace my old ORICO case with the new usb C ORICO case which 100% compatible with raspbian.

If everything when well you should first boot your RPI 4 using your microSD and do not attach your SSD as of yet.

Like usual you can connect to your RPI directly or remotely and in my case I do connect it via SSH (if you unable to connect via SSH then most probably you forget to add `ssh` into the root of your microSD). Remember for the first time raspbian will connect using user `pi` with a default password `raspberry`

Once connected as `pi` you can begin by doing:
```
# update your RPI
sudo apt update -y
sudo apt upgrade -y
sudo rpi-update

# reboot now
sudo reboot now
```

Alright you are now running the latest Raspbian Buster and now we need to update the RPI 4 eeprom. To do that you will need to:
```
# install the rpi-eeprom
sudo apt install rpi-eeprom

# Edit the rpi-eeprom and replace "critical" to "beta"
# FIRMWARE_RELEASE_STATUS="beta"
sudo nano /etc/default/rpi-eeprom-update

# Update/Install the beta eeprom 
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-05-15.bin

# reboot now
sudo reboot now
```

Once your RPI is back online you will be able to check if you are now running the latest eeprom by:
```
# be sure to check that your version is now May 15th
vcgencmd bootloader_version

# be sure to check that your config have BOOTORDER=0xF41
vcgencmd bootloader_config
```

If you are satisfied with all the result then you need to copy some files into your SSD drive. Now you will need to mount the SSD into your RPI 4. Again make sure your SSD is detected by raspbian else it wont work.

```
# begin mounting your SSD Drive
sudo mkdir /mnt/mydisk
sudo mount /dev/sda1 /mnt/mydisk

# copy all files needed for the SSD to boot
sudo cp /boot/*.elf /mnt/mydisk
sudo cp /boot/*.dat /mnt/mydisk

# Now its time to shutdown your RPI
sudo shutdown now
```

Alright you are almost there, so now its time for the result. Remove your microSD and turn on your RPI (make sure your SSD is connected into the USB) Should everything went well you should see that raspbian buster is now up and running without the microSD. Also be sure that `ssh` is on your root SSD directory else you wont be able to do SSH.

Thats all and I hope this guide will help many of you.