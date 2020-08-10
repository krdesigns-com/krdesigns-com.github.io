---
title: Raspberry Pi 4 running Official Ubuntu 20.04 bootable guide from USB SSD Drive
tags: [Ubuntu Official, Raspberry Pi 4, USB bootable drive, guide, ubuntu 20.04 LTS, latest ubuntu, new eeprom, update]
style: border
color: primary
description: Guide to setup Raspberry Pi 4 to boot an official uBuntu 20.04 from your USB SSD (no microSD).
---
Source: [Raspberry Pi Forum](https://www.raspberrypi.org/forums/viewtopic.php?f=131&t=278791), [Raspberry Pi Forum](https://www.raspberrypi.org/forums/viewtopic.php?f=131&t=281152), [Eugene Grechko Website](https://eugenegrechko.com/blog/USB-Boot-Ubuntu-Server-20.04-on-Raspberry-Pi-4), [leepspvideo](https://www.youtube.com/watch?v=SfxFS2mK6ok&t=288s), [Wimpysworld Git](https://github.com/wimpysworld/desktopify), [ETA Prime](https://www.youtube.com/watch?v=zo5eReiXYuo&t=147s)

## Latest update of this guide on 10 August 2020

## Why this guide
I wrote this article to help people who would like to run ubuntu 20.04 server on their Raspberry 4 from USB - SSD Drive rather then from microSD. Also will add bonus on adding ubuntu-mate windows for those who would like to have a desktop enhanced ubuntu-mate.

I try to make this guide as simple as possible so you all can easily make this happen. Furthermore ubuntu is a 64bit and stable version. 

# The Requirements:
- Fresh microSD for eeprom upgrade (for those who have not do this)
- SSD Drive compatible for Linux, or you will need to read [James A. Chambers Guide](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/)
- [Balena Etcher](https://www.balena.io/etcher/) to burn the images
- Latest 32bit [Raspbian OS](https://downloads.raspberrypi.org/raspios_lite_armhf_latest) for microSD
- Latest 64bit [Ubuntu 20.04](https://ubuntu.com/download/raspberry-pi/thank-you?version=20.04&architecture=arm64+raspi) for RPI
- Some files required to be copy for booting

# Upgrade RPI4 eeprom
WARNING YOU WILL NEED TO DO THIS IF YOU HAVE NOT DONE THIS BEFORE! 

First you will need to update your RPI eeprom to the latest stable version. For this step you will require microSD and you can begin by burn your image to the microSD using balena etcher. Once completed you can unplug the microSD and replug back it if you want to run it headleass (Install without Monitor, Keyboard, and Mouse). Headless installation will require that you add `ssh` to microSD boot this way upon booting you can ssh to your RPI.

Now that your RPI is up and running, you will need to find the local IP and you can now SSH to the machine. 
```
# raspbian OS standard password is 'rasberry'
ssh pi@your-local-ip

sudo apt update && sudo apt full-upgrade
sudo reboot now

# Check you are on stable, change it if not
sudo nano /etc/default/rpi-eeprom-update

# update your eeprom
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/stable/pieeprom-2020-07-16.bin
sudo reboot now
```

# Mount SSD drive for boot preparations
Next step you will also need to burn the ubuntu 20.04 image into your SSD Drive using etcher. Once that complete you can also add `ssh` on its boot folder so that you can run your ubuntu headless. 

Next you will need to mount the SSD drive to your RPI (be sure you plug it into the USB3), so you can fixed some files needed to allow your SSD Drive to boot by it self.

This *.elf and *.dat files are required and you can download its from [Raspberry Pi official git REPO](https://github.com/raspberrypi/firmware/tree/master/boot) or you can 

```
# The command to grab the .dat and .elf files
wget $( wget -qO - https://github.com/raspberrypi/firmware/tree/master/boot | perl -nE 'chomp; next unless /[.](elf|dat)/; s/.*href="([^"]+)".*/$1/; s/blob/raw/; say qq{https://github.com$_}' )

# begin mounting your ssd
sudo mkdir /mnt/myboot
sudo mount /dev/sda1 /mnt/myboot

# download files necessaries from https://github.com/raspberrypi/firmware/tree/master/boot
# copy all files need to SSD
sudo cp *.elf /mnt/myboot
sudo cp *.dat /mnt/myboot
```

You will then need to change into root user, so you can perform the next steps

```
# Create root new password
sudo passwd

# Become a root
su

# go to boot directory
cd /mnt/myboot
zcat vmlinuz > vmlinux
```

Next you will need to edit your config.txt in my case I will edit using `nano /mnt/myboot/config.txt`

```
#edit config.txt
[pi4]
max_framebuffers=2
dtoverlay=vc4-fkms-v3d
boot_delay
kernel=vmlinux
initramfs initrd.img followkernel
```

Now you will need to make an `auto_decompress_kernel` script in your boot SSD Drive and please make sure you change user permission `chmod +x auto_decompress_kernel`. The script need to be written down at this directory `/mnt/myboot` you can use `nano /mnt/myboot/auto_decompress_kernel` and paste the below scripts.
```
#!/bin/bash -e

#Set Variables
BTPATH=/boot/firmware
CKPATH=$BTPATH/vmlinuz
DKPATH=$BTPATH/vmlinux

#Check if compression needs to be done.
if [ -e $BTPATH/check.md5 ]; then
	if md5sum --status --ignore-missing -c $BTPATH/check.md5; then
		echo -e "\e[32mFiles have not changed, Decompression not needed\e[0m"
		exit 0
	else echo -e "\e[31mHash failed, kernel will be compressed\e[0m"
	fi
fi

#Backup the old decompressed kernel
mv $DKPATH $DKPATH.bak

if [ ! $? == 0 ]; then
	echo -e "\e[31mDECOMPRESSED KERNEL BACKUP FAILED!\e[0m"
	exit 1
else echo -e "\e[32mDecompressed kernel backup was successful\e[0m"
fi

#Decompress the new kernel
echo "Decompressing kernel: "$CKPATH".............."

zcat $CKPATH > $DKPATH

if [ ! $? == 0 ]; then
	echo -e "\e[31mKERNEL FAILED TO DECOMPRESS!\e[0m"
	exit 1
else echo -e "\e[32mKernel Decompressed Succesfully\e[0m"
fi

#Hash the new kernel for checking
md5sum $CKPATH $DKPATH > $BTPATH/check.md5

if [ ! $? == 0 ]; then
	echo -e "\e[31mMD5 GENERATION FAILED!\e[0m"
else echo -e "\e[32mMD5 generated Succesfully\e[0m"
fi

#Exit
exit 0
```

You will need to make the kernel boot script. `nano /etc/apt/apt.conf.d/999_decompress_rpi_kernel`
and again you will need to `chmod +x /etc/apt/apt.conf.d/999_decompress_rpi_kernel`

```
# content 999_decompress_rpi_kernel
DPkg::Post-Invoke {"/bin/bash /boot/firmware/auto_decompress_kernel"; };
```

Its DONE, you should now be able to boot from your SSD Drive without the need of your microSD anymore. Once you complete the boot be sure you run `/boot/firmaware/auto_decompress_kernel` once and you are set.


# Bonus trick for adding an Optimized UBUNTU-MATE windows for those who would like to used it as a Desktop
```
git clone https://github.com/wimpysworld/desktopify.git
cd desktopify
./desktopify --de ubuntu-mate
```

Remember this process will take some times to complete. So now sit back relax and make sure you hookup your RPI4 to your monitor, keyboard, and mouse. For those who own an ultrawide monitor like mine this setup will automatically setup to **"2560x1080"** which very nice indeed. Enjoy!