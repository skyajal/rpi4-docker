# Raspberry Pi 4 - Docker Setup with Raspbian OS 64 bit

This guide contains several options to configure docker on raspbian OS 64 bit. This guide is not complete at this time.

## **Quick Setup**
**_Tip_:** Below are some recommendations to do prior to installing docker.
   - Run the commands `sudo apt update` and `sudo apt upgrade` to bring the OS up-to-date.
   - Check that the timezone is set to youur region.  Use the command `date` to view the current settings. Run `sudo raspi-config` to set the timezone for your region.
   - Check that the locale is set to your region. Use the command `locale` to view the current settings. Run `sudo raspi-config` to set the locale for your region. Reboot for the changes to take affect.

The easiest method to setup docker on raspbian OS 64 bit is to use the [conveinence script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script). After the installation is complete, add the pi user to the docker group to run docker commands directly. Run the command `sudo usermod -aG docker pi` then log out and log back in for the changes to take affect. If you do not plan to use a [user-defined bridge](https://docs.docker.com/network/bridge/) or a [macvlan](https://docs.docker.com/network/macvlan/), docker is now ready to use.

## **User-defined bridge**
placeholder

## **Macvlan**
Add a [macvlan](https://docs.docker.com/network/macvlan/) network to docker by selecting an ip range not being used by DHCP. If a bridge is used, replace eth0 with br0 or the name of the bridge interface. This step can only be done after docker is installed and running. See the example below.
```
docker network create -d macvlan -o parent=eth0 \
--subnet 192.168.0.0/24 --gateway 192.168.0.1 \
--ip-range 192.168.0.88/30 macvlan
```
In this example, the subnet 192.168.0.0/24 is used to match the current network that is being used. The default gateway is also specified. The ip range defines which ip addresses will be available to docker to use. Setting the subnet mask to /30 will allow the macvlan to use the ip range `192.168.0.88 - 192.168.0.91`. Docker will assign an ip adress automatically or an ip adress can be set statically.

**_Tip_:** The above example will allow a container to be reachable outside the host. It will not allow communication between the host and the container. For containers to be able to communicate with the host, additional network settings must be added. See below for details.

### **(Optional)** Add configuration for systemd-networkd to provide a macvlan bridge for docker. 
   If a bridge is being used, create a new file named `/etc/systemd/network/br0.network` using the editor of your choice. The file should contain the following settings:
   ```
   [Match]
   Name=br0

   [Network]
   MACVLAN=mac0
   ```
   If a bridge is not being used, create a new file named `/etc/systemd/network/eth0.network` using the editor of your choice. The file should contain the following settings:
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
   Address=192.168.0.88/24
   Gateway=192.168.0.1
   DNS=192.168.0.1
   DNS=1.1.1.1
   Domains=example.net
   ```
   **_Tip_:** The above example is for setting a static ip. The ip address should be a part of the range used for the macvlan. See step 4 for additional information.
   - If a bridge is not being used, enable systemd-networkd when the system starts: `sudo systemctl enable systemd-networkd`.

## **(Optional)** Create a bridged interface on the host. 
   This can be handy if you plan on using other services such as qemu along with docker. Create the file `/etc/systemd/network/br0.netdev` using the editor of your choice. The file should contain the following settings:
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
   Finally, edit the file `/etc/dhcpcd.conf` using the editor of your choice. Add the following settings to the file:
   ```
   # Deny DHCP to the following interfaces
   denyinterfaces eth0 wlan0

   # Configure the bridge
   interface br0
   ```
   **_Tip_:** If DHCP is being used, place both settings at the end of the file. Otherwise put the `denyinterfaces eth0 wlan0` before any `interface` setting. A static ip address can be configured if desired. See the example provided in the file.

   **_Tip_:** It is recommended to disable the wireless interface by adding the following option to `/boot/config.txt` then wlan0 is not needed in the denyinterfaces noted above.
   ```
   dtoverlay=disable-wifi
   ```
   Once these steps are complete, it is recommended to reboot to test the changes. If DHCP is used, the ip address may of changed. If a static ip is used, check to see if the ip address can be pinged.

   - If the optional steps above are not used, edit `/etc/dhcpdcp.conf` to configure a static ip address if desired. See the example provided in the file. Otherwise, DHCP will be used.

Install docker using [installation instructions](https://docs.docker.com/engine/install/debian/) provided for installing the [conveinence script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script). At this point docker is now installed. If you do not plan to create macvlans or bridge networks, you can stop here and start using docker. Below are the optional steps to create macvlans and bridge networks for docker.

**_Tip_:** It is recommended to add the pi user to the docker group to run docker commands directly. Run the command `sudo usermod -aG docker pi` then log out and log back in for the changes to take affect.

**(Optional)** Add a [macvlan](https://docs.docker.com/network/macvlan/) network to docker by selecting an ip range not being used by DHCP. The optional steps for creating the configuration files for `systemd-networkd` must be done 1st in order to use macvlan. If a bridge is used, replace eth0 with br0. This step can only be done after docker is installed and running. See the example below.
```
docker network create -d macvlan -o parent=eth0 \
--subnet 192.168.0.0/24 --gateway 192.168.0.1 \
--ip-range 192.168.0.88/30 macvlan
```
In this example, the subnet 192.168.0.0/24 is used to match the current network that is being used. The default gateway is also specified. The ip range defines which ip addresses will be available to docker to use. Setting the subnet mask to /30 will allow the macvlan to use the ip range `192.168.0.88 - 192.168.0.91`. 