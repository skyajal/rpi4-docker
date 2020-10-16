# Raspberry Pi 4 - Docker Setup
This document descibes the steps used to create a docker environment on a Raspberry Pi 4 using a 2 Terabyte SSD without a microsd card.
In order to boot from a SSD or USB device, follow the steps outlined at [krdesigns.com](https://krdesigns.com/articles/Boot-raspbian-ubuntu-20.04-official-from-SSD-without-microsd) to setup your SSD or USB device.

After Ubuntu has been successfully installed onto your device, allow it to complete unattended updates and login to change the password. Install any additional updates that are needed.

There is some additional things I like to change before installing docker. These are completely optional.
1. Rename the default ubuntu username
2. Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
3. Disable the sdcard status (green) light

## Renaming the default ubuntu username
I prefer to login with my username rather than ubuntu. Why change it though? It's a personal preference to keep my user id and group id the same across all of my devices.

**Note:** You will need root privileges to make these changes. You can either login directly as root or via ssh if you prefer.

Run the following commands as root:
```
usermod -l <username> ubuntu
usermod -m -d /home/<username> <username>
groupmod -n <username> ubuntu
```
Replace \<username\> with your desired username.

## Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
The original script provided by Argon40 doesn't have support for Ubuntu. Luckily, [meuter/argon-one-case-ubuntu-20.04](https://github.com/meuter/argon-one-case-ubuntu-20.04) has provided an updated script to do just that.
Follow his instructions if you would like to use this with your Argon ONE case.
