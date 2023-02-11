# RaspberryPi 4 - Docker Setup with Raspbian OS 64 bit

This guide contains several options to configure docker on raspbian OS 64 bit. This guide is provided as-is. Use at your own risk. This guide does not go into detail of setting up containers after the initial installation is complete. This is only for the installation of docker. Building containers via dockerfile or docker-compose is not covered here. There are plenty of guides online that discusses this topic.

## **Quick Setup**
**_Tip_:** Below are some recommendations to do prior to installing docker.
- Run the commands `sudo apt update` and `sudo apt upgrade` to bring the OS up-to-date.
- Check that the timezone is set to your region.  Use the command `date` to view the current settings. Run `sudo raspi-config` to set the timezone for your region.
- Check that the locale is set to your region. Use the command `locale` to view the current settings. Run `sudo raspi-config` to set the locale for your region. Reboot for the changes to take affect.
- This guide assumes only ethernet will be used. It is recommended to disable the wireless interface if it will not be used by adding the following option to `/boot/config.txt`.
```
dtoverlay=disable-wifi
```

The easiest method to setup docker on raspbian OS 64 bit is to use the [convenience script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script). After the installation is complete, add your user to the docker group to run docker commands directly. Run the command `sudo usermod -aG docker (your_username)` then log out and log back in for the changes to take affect. If you do not plan to use a [user-defined bridge](https://docs.docker.com/network/bridge/) or a [macvlan](https://docs.docker.com/network/macvlan/), docker is now ready to use.

## **User-defined bridge**
Use the following to create a [user-defined bridge](https://docs.docker.com/network/bridge/). The default bridge `docker0` in docker does not allow containers to connect each other via container names used as dns hostnames. Therefore, it is recommended to create a user-defined bridge and attach the containers to that bridge. Change the bridge name, subnet and gateway ip to the desired setting. See the example below.
```
docker network create -d bridge --subnet 192.168.10.0/24 \
--gateway 192.168.10.1 \
-o "com.docker.network.bridge.host_binding_ipv4"="0.0.0.0" \
-o "com.docker.network.bridge.enable_ip_masquerade"="true" \
-o "com.docker.network.bridge.enable_icc"="true" \
-o "com.docker.network.bridge.name"="armnet" \
-o "com.docker.network.driver.mtu"="1500" armnet
```

#### **(Optional)** Disable the default bridge
To disable the default bridge `docker0`, create a new file`/etc/docker/daemon.json` with the following setting. Do not do this if you want to keep
the default docker bridge. Please note this step is optional and it assumes you have created a user-defined bridge as noted above this section. If you have not created or do not want a user-defined bridge, skip this step. If daemon.json already exists, be sure to add commas as needed for the json config.
```json
{
   "bridge": "none"
}
```
## **Macvlan**
Add a [macvlan](https://docs.docker.com/network/macvlan/) network to docker by selecting an ip range not being used by DHCP or by other hosts. If a host bridge is used, replace eth0 with br0 or with the name of the bridge interface. The subnet 192.168.0.0/24 is used to match the current network that is being used. The default gateway is also specified. The ip range defines which ip addresses will be available for docker to use. Setting the subnet mask to /30 will allow the macvlan to use the ip range `192.168.0.88 - 192.168.0.91`. Docker will assign an ip address automatically or an ip address can be set statically. Lastly, name the network as `macvlan` or to the desired name. See the example below.
```
docker network create -d macvlan -o parent=eth0 \
--subnet 192.168.0.0/24 --gateway 192.168.0.1 \
--ip-range 192.168.0.88/30 macvlan
```

**_Tip_:** The above example will allow a container to be reachable outside the host. It will not allow communication between the host and the container. For containers to be able to communicate with the host, additional network settings must be added. See below for details.

#### **(Optional)** Add configuration for systemd-networkd to provide a macvlan bridge for docker. 
If a host bridge is being used, create a new file named `/etc/systemd/network/br0.network` using the editor of your choice. The file should contain the following settings:
```
[Match]
Name=br0

[Network]
MACVLAN=mac0
```

If a host bridge is not being used, create a new file named `/etc/systemd/network/eth0.network` using the editor of your choice. The file should contain the following settings:
```
[Match]
Name=eth0

[Network]
MACVLAN=mac0
```

Next, create a new file named `/etc/systemd/network/mac0.netdev` using the editor of your choice. The file should contain the following settings:
```
[NetDev]
Name=mac0
Kind=macvlan

[MACVLAN]
Mode=bridge
```
Then create a new file named `/etc/systemd/network/mac0.network` using the editor of your choice. The file should contain the following settings:
```
[Match]
Name=mac0

[Network]
DHCP=no
ipForward=yes
LinkLocalAddressing=no
Address=192.168.0.87/24
Gateway=192.168.0.1
DNS=192.168.0.1
DNS=1.1.1.1
Domains=example.net
```
The above example uses a static ip. The ip address will act as the bridge used for the macvlan. Set the ip addresses to the desired settings. Change the domain to the desired domain name.

**_IMPORTANT_:** If a host bridge is not being used, enable systemd-networkd when the system starts: `sudo systemctl enable systemd-networkd`. Edit the file `/etc/dhcpcd.conf` using the editor of your choice. Add the following settings to the file.
```
# Deny DHCP to the following interfaces
denyinterfaces mac0

# Configure the interface
interface eth0
```
**_Tip_:** If DHCP is being used, place both settings at the end of the file. Otherwise put the `denyinterfaces mac0` before any `interface` setting. A static ip address can be configured if desired. See the example provided in the file.

## **(Optional) Create a bridge on the host.** 
This can be handy if you plan on using other services such as qemu along with docker. Create the file `/etc/systemd/network/br0.netdev` using the editor of your choice. The file should contain the following settings.
```
[NetDev]
Name=br0
Kind=bridge
```
Next, create a file named `/etc/systemd/network/eth0.network` using the editor of your choice. The file should contain the following settings:
```
[Match]
Name=eth0

[Network]
Bridge=br0
```
Now enable systemd-networkd when the system starts: `sudo systemctl enable systemd-networkd` This will create and populate the bridge interface during boot.
Finally, edit the file `/etc/dhcpcd.conf` using the editor of your choice. Add the following settings to the file.
```
# Deny DHCP to the following interfaces
denyinterfaces eth0 mac0

# Configure the interface
interface br0
```
**_Tip_:** If DHCP is being used, place both settings at the end of the file. Otherwise put the `denyinterfaces eth0 mac0` before any `interface` setting. A static ip address can be configured if desired. See the example provided in the file. Remove `mac0` if you do not use macvlan bridge.
