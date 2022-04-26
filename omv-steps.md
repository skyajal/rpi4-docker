!!Work In Progress!!
***
 How to setup KVM and Docker on OMV 6
***

Install latest raspios lite 64 OS from https://downloads.raspberrypi.org/raspios_lite_arm64/images/ and then use the following commands, executed one at at time:

sudo apt-get update

sudo apt-get upgrade -y

sudo rm -f /etc/systemd/network/99-default.link

When all three commands above are complete, type;

sudo reboot

Use OMV 6 script:

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
