**Please note the information below is outdated. I will revamp it when time permits.**

# Raspberry Pi 4 - Docker Setup with Ubuntu 20.04 LTS
This document describes the steps that were used to create a docker environment on a Raspberry Pi 4 using a SSD without a microsd card.
In order to boot from a SSD, follow the steps outlined at [krdesigns.com](https://krdesigns.com/articles/Boot-raspbian-ubuntu-20.04-official-from-SSD-without-microsd).

Here are some options that are suggested to change before installing docker. All of the steps are completely optional.
1. Rename the default ubuntu username
2. Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
3. Disable the microsd status (green) light
4. Disable wireless and bluetooth
5. Install Network Manager for macvlan bridge support
6. Set the hostname, timezone and NTP servers
7. Do a final update check before rebooting

## Rename the default ubuntu username
I prefer to login with my username rather than ubuntu. Why change it though? It's a personal preference to keep my user id and group id the same across all of my devices. This can be done with the [cloud-config](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) settings before initial boot. To keep it simple without using with the cloud-config, Here are some basic commands to do this.

**Note:** Root privileges are required to make these changes. Login as root on the Raspberry Pi or via ssh. Do not use su while logged into the ubuntu user.

Run the following commands as root:
```
usermod -l <username> ubuntu
usermod -m -d /home/<username> <username>
groupmod -n <username> ubuntu
```
Replace \<username\> with your desired username.

## Install argon1.sh script for the Argon ONE Pi 4 Raspberry Pi Case
The original script provided by Argon40 doesn't have support for Ubuntu. Luckily, [meuter/argon-one-case-ubuntu-20.04](https://github.com/meuter/argon-one-case-ubuntu-20.04) has provided an updated script to do just that. Follow the instructions to use the script with the Argon ONE case.

## Disable the microsd status \(green\) light
This is purely cosmetic. Booting from SSD renders this light no longer useful. Edit `/boot/firmware/config.txt` and add the following under the `[pi4]` tag.
```
# Disable the Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off
```

## Disable wireless and bluetooth
Wireless and bluetooth are not needed. To disable them during boot, edit `/boot/firmware/config.txt` and add the following under the `[all]` tag.
```
dtoverlay=disable-bt
dtoverlay=disable-wifi
```
## Install Network Manager for macvlan bridge support
Netplan doesn't support macvlans natively. Initial testing to setup a macvlan with netplan using networkd failed to survive a reboot. Going back to using ifupdown was not an option. Use the script example found in the [bug #1664847](https://bugs.launchpad.net/netplan/+bug/1664847) to setup a macvlan bridge so the Raspberry Pi can communicate with containers using macvlan. If macvlan bridging will not be used then this section can be skipped.

After installing the `network-manager` package, there is some additional steps to do before rebooting.
1. Add the script mentioned in the bug above as a workaround to allow macvlan bridging. Edit `/etc/NetworkManager/macvlan.sh` and add the following example:
```
#!/usr/bin/bash

IFACE=${1}
ACTION=${2}

if [[ "${IFACE}" == "br0" && "${ACTION}" == "up" ]]; then
        ip link add link br0 name macvlan0 type macvlan mode bridge
        ip addr add 172.29.10.224/32 dev macvlan0
        ip link set macvlan0 up
        ip route add 172.29.10.224/27 dev macvlan0
fi

if [[ "${IFACE}" == "br0" && "${ACTION}" == "pre-down" ]]; then
        ip route del 172.29.10.224/27
        ip link set macvlan0 down
        ip link del dev macvlan0
fi
```
Change the IP addresses to match the desired settings. Set the appropriate ownership/permissions and symlink to the hook locations.
```
chown root:root /etc/NetworkManager/macvlan.sh
chmod 700 /etc/NetworkManager/macvlan.sh
ln -s /etc/NetworkManager/macvlan.sh /etc/NetworkManager/dispatcher.d/pre-up.d/macvlan.sh
ln -s /etc/NetworkManager/macvlan.sh /etc/NetworkManager/dispatcher.d/pre-down.d/macvlan.sh
ln -s /etc/NetworkManager/macvlan.sh /etc/NetworkManager/dispatcher.d/no-wait.d/macvlan.sh
ln -s /etc/NetworkManager/macvlan.sh /etc/NetworkManager/dispatcher.d/macvlan.sh
```

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

## Do a final update check before rebooting
Run the following command as root or using sudo: `apt get update && apt get upgrade -y` and then finally reboot. The Raspberry Pi should now be ready to install docker. Verify ssh is accessible to continue. If not, check back through the previous steps.

## Install Docker
Use the steps outlined on the [docker installation guide](https://docs.docker.com/engine/install/ubuntu/) or alternatively use the provided [package](https://packages.ubuntu.com/focal/docker.io) in the Ubuntu repository. Please only choose one.
  - Check to make sure the username is part of the docker group. This can be checked with the following command: `id <username> |grep docker`. If not, use the following command: `usermod -aG docker <username>`. This will add the user to the docker group to run commands as a non-root user. See the [docker documentation](https://docs.docker.com/engine/install/linux-postinstall/) for more details.

## Install docker-compose
Installing docker-compose can be done using several methods. The easiest method is to install the `docker-compose` package. However, this version is behind the latest release. This is not a bad thing either. Alternative methods include using [linuxserver.io](https://hub.docker.com/r/linuxserver/docker-compose) container image or the [binary](https://github.com/linuxserver/docker-docker-compose) available for arm64 can also be used. In this example, the binary file will be used. Use the following commands as root or using sudo:
```
wget https://github.com/linuxserver/docker-docker-compose/releases/download/1.27.4-ls17/docker-compose-arm64
mv docker-compose-arm64 docker-compose && mv docker-compose /usr/local/bin
chown root. /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose
```
Test the version by running the following command: `docker-compose --version`

## Docker macvlan
Use the following to create a macvlan for the stack to use. Replace the parent
device and networking information. Below is an example to use IP addresses from
a network range not used by dhcp. I opted not to create the network settings
in the compose file. Additional information: https://docs.docker.com/network/macvlan
```
docker network create -d macvlan -o parent=eth0 \
-o "com.docker.network.macvlan.name"="macvlan" \
--subnet 192.168.0.0/24 --gateway 192.168.0.1 \
--ip-range 192.168.0.88/30 macvlan
```
**Note:** The option -o "com.docker.network.macvlan.name"="macvlan" is optional and not needed.

To see a list of IPs that will be available for docker to assign, use this
IP Calculator: http://jodies.de/ipcalc

Network:   192.168.0.88/30 \
Broadcast: 192.168.0.91 \
HostMin:   192.168.0.89 \
HostMax:   192.168.0.90

Even though the range only shows 2 IPs, all of the IPs can be used since the
containers will use the host subnet. In this example, /24 is used for the host.

To allow the host running the containers network access, use the following
settings to create a bridge. Either use sudo or login as root and run these
commands.

ip link add mac0 link eth0 type macvlan mode bridge \
ip addr add 192.168.0.88/30 dev mac0 \
ip link set up mac0

Note: If you are using Netplan, there is a known bug https://bugs.launchpad.net/netplan/+bug/1664847 which doesn't support macvlan
after a reboot. Please see https://gist.github.com/timcharper/d547fbe13bdd859f4836bfb02197e295 and https://gist.github.com/jooter/8f95555021d2a4e7cb6241368473ed55 for example workarounds with networkd. I tried this method but was unable to get it to retain after
a reboot. Using the Network Manager method mentioned in the bug works. While not ideal, there seems to be no good implementation documented for going the networkd method. Going back to using ifupdown from the Ubuntu archive was not an option either I was willing to explore.

## Docker bridging
Use the following to create a bridge for the stack to use. The default bridge
network in docker does not allow containers to connect each other via container
names used as dns hostnames. Therefore, it is recommended to first create a user
defined bridge network and attach the containers to that network. This must also
be done prior to using docker-compose. I opted not to create the network settings
in the compose file. Change the bridge name to whatever you want along with the
subnet and gateway IP. Keep armnet the same or replace all instances
with the name you want to use. See The example below:

docker network create --d bridge --subnet 192.168.10.0/24 \
--gateway 192.168.10.1 \
-o "com.docker.network.bridge.host_binding_ipv4"="0.0.0.0" \
-o "com.docker.network.bridge.enable_ip_masquerade"="true" \
-o "com.docker.network.bridge.enable_icc"="true" \
-o "com.docker.network.bridge.name"="armnet" \
-o "com.docker.network.driver.mtu"="1500" armnet

## Naming the stack
To name the stack without using the directory name, create .env file in the same
directory the compose file is located and add the following:
COMPOSE_PROJECT_NAME=mystack
Alternatively, use -p "mystack" before up.
Example: docker-compose -p "mystack" up -d

## Docker compose file example
This example uses version 3.8 of the [compose](https://docs.docker.com/compose/compose-file) file format.
