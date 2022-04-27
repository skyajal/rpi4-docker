## Documentation updating in progress! Use at your own risk.
***
 How to setup KVM and Docker on OMV 6
***
This guide/howto will walk you through a new installation of [openmediavault](https://www.openmediavault.org/) 6. Then we will add support for KVM and Docker to run together over a bridged network interface. Some knowledge of configuring networking on raspberry pi is recommended.

### Install OS
Install the latest raspios lite 64 OS image from https://downloads.raspberrypi.org/raspios_lite_arm64/images/ to your SD card or USB drive flashed with the latest arm64 image. As of this guide, version: [2022-04-04-raspios-bullseye-arm64-lite.img.xz](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-04-07/2022-04-04-raspios-bullseye-arm64-lite.img.xz),[torrent](https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2022-04-07/2022-04-04-raspios-bullseye-arm64-lite.img.xz.torrent) are used in the examples below. After you have flashed either an SD card or USB drive, follow the inital setup. Once you are able to login via console or ssh, run the following commands:
```
sudo apt-get update

sudo apt-get upgrade -y

sudo rm -f /etc/systemd/network/99-default.link
```
When all three commands above are complete, type `sudo reboot`

### Install OMV 6 via script:
Run the following command to install OMV 6.
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash


// For later
Go into home > network > interfaces
Select eth0 and then delete by clicking yes

Create new bridge for omv to use:

http://omv.local/#/network/interfaces/bridge/create

Install Docker from omv-extras

Install kvm plugin

add user to libvirt group. This will eliminate the need for sudo.

Optional: install virt-manager on client machine to assist with managing kvm.

To list current network settings. Below shows the default which is virbr0.

pi@omv:~ $ sudo virsh net-list --all
 Name      State      Autostart   Persistent
----------------------------------------------
 default   inactive   no          yes

create host-bridge.xml containing the following:

<network>
  <name>host-bridge</name>
  <forward mode="bridge" />
  <bridge name="br0" />
</network>

Add it: pi@omv:~ $ virsh net-define host-bridge.xml

pi@omv:~ $ virsh net-list --all
 Name          State      Autostart   Persistent
--------------------------------------------------
 host-bridge   inactive   no          yes

pi@omv:~ $ virsh net-start host-bridge
Network host-bridge started

pi@omv:~ $ virsh net-autostart host-bridge
Network host-bridge marked as autostarted

sudo iptables -A FORWARD -i br0 -o br0 -j ACCEPT

sudo apt install -y iptables-persistent  then follow the prompts when installing the package.

<<Work In Progress>>
