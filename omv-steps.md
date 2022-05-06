### About this guide
Hello and welcome. I am [shadowzero](https://forum.openmediavault.org/wsc/index.php?user/2842-shadowzero/) from the [OMV forums](https://forum.openmediavault.org/). I used to contribute to openmediavault some time ago. After OMV 3 was released, my priorities changed. Since then, I have contributed in testing the [omv-extras](https://wiki.omv-extras.org/) kvm plugin on the Raspberry Pi 4. I still think OMV is awesome. I just don't use it on a regular basis anymore. I prefer building my own "GUI/Web Interface" based on docker containers and kvm while retaining most activities to CLI based functions. Aside from my guides to do this without OMV, I still like to show this project some love by contributing when I can.
***
 How to setup KVM and Docker on OMV 6
***
This guide/howto will walk you through a new installation of [openmediavault](https://www.openmediavault.org/) 6. Then we will add support for kvm and docker to run together over a bridged network interface. Some knowledge of configuring networking on raspberry pi is recommended.

### Install OS
Install the latest raspios lite 64 OS image from https://downloads.raspberrypi.org/raspios_lite_arm64/images/ to your SD card or USB drive flashed with the latest arm64 image. As of this guide, version: [2022-04-04-raspios-bullseye-arm64-lite.img.xz](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-04-07/2022-04-04-raspios-bullseye-arm64-lite.img.xz),[torrent](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-04-07/2022-04-04-raspios-bullseye-arm64-lite.img.xz.torrent) are used in the examples below. After you have flashed either an SD card or USB drive, follow the inital setup. Once you are able to login via console or ssh, run the following commands:
```
sudo apt-get update

sudo apt-get upgrade -y

sudo rm -f /etc/systemd/network/99-default.link
```
When all three commands above are complete, type `sudo reboot` and log back into your device.

### Install OMV 6 via script:
Run the following command to install OMV 6:
`wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash`
### Setup the bridge:
After the OMV install is complete, navigate to: `home > network > interfaces`. Select `eth0` and then `delete` by clicking `yes`.
Add a new bridge interface named `br0` that uses `eth0`. Reboot after making the changes. 
### Install the docker and kvm plugins
See the steps below:
- Install docker from omv-extras.
- Install kvm plugin from omv-extras.
  - Add your user to libvirt and kvm groups. This will eliminate the need for sudo.
  - Optional: install virt-manager on a client machine to manage kvm.

### Setup kvm to use the bridge
To list current network settings, run the commad below. This example shows the default which is virbr0.
```
pi@omv:~ $ virsh net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes
```
Create the file `host-bridge.xml` and add the the following:
```
<network>
  <name>host-bridge</name>
  <forward mode="bridge" />
  <bridge name="br0" />
</network>
```
Next, we will add it by executing the following command: `pi@omv:~ $ virsh net-define host-bridge.xml`.
Check the network settings again and you should now see the `host-bridge` has been added. See the example below.
```
`pi@omv:~ $ virsh net-list --all`
 virsh net-list --all
 Name          State      Autostart   Persistent
--------------------------------------------------
 default       inactive   no          yes
 host-bridge   active     no          yes
```
Start the host-bridge with the following command:
```
pi@omv:~ $ virsh net-start host-bridge
Network host-bridge started
```
We will want the host-bridge to start automatically. Run the following command:
```
pi@omv:~ $ virsh net-autostart host-bridge
Network host-bridge marked as autostarted
```
We need to set iptables to allow fowarding on the bridge network `br0`. This will allow kvm and docker to use the same bridge interface.
Run the following command:`sudo iptables -A FORWARD -i br0 -o br0 -j ACCEPT`

Lastly, we need to make the changes persistent. Run the following command:
`sudo apt install -y iptables-persistent` then follow the prompts when installing the package.
You should now be able to run kvm and docker together on OMV 6. Enjoy! :)
