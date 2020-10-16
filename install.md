# Raspberry Pi 4 - Docker Setup
This document descibes the steps used to create a docker environment on a Raspberry Pi 4 using a 2 Terabyte SSD without a microsd card.
In order to boot from a SSD or USB device, follow the steps outlined at [krdesigns.com](https://krdesigns.com/articles/Boot-raspbian-ubuntu-20.04-official-from-SSD-without-microsd) to setup your SSD or USB device.

After Ubuntu has been successfully installed onto your device, allow it to complete unattended updates and login to change the password. Install any additional updates that are needed.

There is some additional things I like to change before installing docker. These are completely optional.
1. Rename the default ubuntu username
2. Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
3. Disable the microsd status (green) light
4. Install Network Manager for macvlan bridge support

## Renaming the default ubuntu username
I prefer to login with my username rather than ubuntu. Why change it though? It's a personal preference to keep my user id and group id the same across all of my devices. You can also modify the [cloud-config](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) settings before initial boot to accomplish the same thing. To keep it simple without messing with the cloud-config, I use some basic commands to do this.

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

## Disable the microsd status \(green\) light
This is purely cosmetic. Since I am booting from SSD, this light is no longer useful. Edit /boot/firmware/config.txt and add the following under the \[pi4\] tag
```
# Disable the Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off
```
## Install Network Manager for macvlan bridge support
Netplan doesn't support macvlans natively. I tried to setup a macvlan in netplan using networkd but I could not get it to survive a reboot. Going back to ifupdown was also not an option I wanted to try. I use the script example found in the [bug #1664847](https://bugs.launchpad.net/netplan/+bug/1664847) to setup a macvlan bridge so the Raspberry Pi can communicate with containers using macvlan.
If you don't plan to use macvlans then you can skip this step.

After you have installed the network-manager package, there is some additional steps to do before rebooting.
1. Add the script mentioned in the bug above as a workaround to allow macvlan bridging.
2. Edit /etc/netplan/50-cloud-init.yaml and add the renderer for NetworkManager.
   - If you plan to use dhcp, this is the only change needed.
   - If you plan to use a static IP then set the desired settings. See the example below:
   ```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eth0:
      addresses:
        - 10.10.10.2/24
      gateway4: 10.10.10.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [10.10.10.1, 1.1.1.1]
```
Also add /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with network: {config: disabled}

**Optional:** If you do not want Network Manager changing /etc/resolv.conf add dns=none to /etc/NetworkManager/NetworkManager.conf under the \[main\] section. This also means you can disable the systemd-resolved daemon as it is no longer being used.
