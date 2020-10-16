# Raspberry Pi 4 - Docker Setup
This document descibes the steps used to create a docker environment on a Raspberry Pi 4 using a 2 Terabyte SSD without a microsd card.
In order to boot from a SSD or USB device, follow the steps outlined at [krdesigns.com](https://krdesigns.com/articles/Boot-raspbian-ubuntu-20.04-official-from-SSD-without-microsd) to setup your SSD or USB device.

After Ubuntu has been successfully onto your device, allow it to complete unattended updates and login to change the password. Install any additional updates that are needed.

There is some additional things I like to change before installing docker. These are completely optional.
1. Rename the default ubuntu username
2. Install argon1 script for argon40 case
3. Disable the sdcard status (green) light
