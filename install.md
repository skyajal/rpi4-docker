# Raspberry Pi 4 - Docker Setup
This document describes the steps that were used to create a docker environment on a Raspberry Pi 4 using a SSD without a microsd card.
In order to boot from a SSD, follow the steps outlined at [krdesigns.com](https://krdesigns.com/articles/Boot-raspbian-ubuntu-20.04-official-from-SSD-without-microsd).

Here are some options I prefer to change before installing docker. All of the steps are completely optional.
1. Rename the default ubuntu username
2. Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
3. Disable the microsd status (green) light
4. Disable wireless and bluetooth
5. Install Network Manager for macvlan bridge support
6. Set the hostname, timezone and NTP servers

## Renaming the default ubuntu username
I prefer to login with my username rather than ubuntu. Why change it though? It's a personal preference to keep my user id and group id the same across all of my devices. This can be done with the [cloud-config](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) settings before initial boot. To keep it simple without using with the cloud-config, Here are some basic commands to do this.

**Note:** You will need root privileges to make these changes. You can either login directly as root or via ssh if you prefer. You cannot su while logged into the ubuntu user.

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
This is purely cosmetic. Since I am booting from SSD, this light is no longer useful. Edit `/boot/firmware/config.txt` and add the following under the `[pi4]` tag
```
# Disable the Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off
```

## Disable wireless and bluetooth
We don't need wireless nor bluetooth so let's disable them during boot. Edit `/boot/firmware/config.txt` and add the following under the `[all]` tag
```
dtoverlay=disable-bt
dtoverlay=disable-wifi
```
## Install Network Manager for macvlan bridge support
Netplan doesn't support macvlans natively. I tried to setup a macvlan in netplan using networkd but I could not get it to survive a reboot. Going back to ifupdown was also not an option I wanted to try. I use the script example found in the [bug #1664847](https://bugs.launchpad.net/netplan/+bug/1664847) to setup a macvlan bridge so the Raspberry Pi can communicate with containers using macvlan.
If you don't plan to use macvlans then you can skip this step.

After you have installed the `network-manager` package, there is some additional steps to do before rebooting.
1. Add the script mentioned in the bug above as a workaround to allow macvlan bridging.
2. Edit /etc/netplan/50-cloud-init.yaml and add the renderer for NetworkManager.
   - If you plan to use dhcp, this is the only change needed.
   - If you plan to use a static IP then set the desired settings. See the example below.
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

3. Create a new file named `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg` and add the following: `network: {config: disabled}`
4. **Optional:** If you do not want Network Manager changing `/etc/resolv.conf` add `dns=none` to /etc/NetworkManager/NetworkManager.conf under the `[main]` section. You can then disable the systemd-resolved daemon as it is no longer being used. Use the following command as root or using sudo: `systemctl disable systemd-resolved`. You will also need to remove the old symbolic link and create a new `/etc/resolv.conf` file with your nameservers.
5. Disable networkd as it will no longer be used after rebooting. Use the following command as root or using sudo: `systemctl disable systemd-networkd`

## Set the hostname, timezone and NTP servers
1. Set the hostname using the following command as root or using sudo: `hostanmectl set-hostname <hostname>`. Replace <hostname> with your desired name.
2. Set the timezone using the following command as root or using sudo: `timedatectl set-timezone <timezone>`. Replace <timezone> with your desired timezone.
3. Set the NTP servers to be used by editing `/etc/systemd/timesyncd.conf` file. Uncomment `NTP=` and add your NTP server. Optionally, you can set a fallback NTP server.
