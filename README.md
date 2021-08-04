# Raspberry Pi 4 - Docker Setup with Raspbian OS 64 bit

Below is the steps that were used to setup docker on Raspbian OS 64 bit. Detailed information will be added as time permits.

1. Install Raspbian OS 64 bit on the desired storage of your choice. Once this step has been completed, here are a few things to do before moving on to the next step:
   - Run the commands `apt update` and `apt upgrade` to bring the OS up-to-date.
   - Check that the timezone is set correctly by running the `date` command. Run `raspi-config` to set the timezone.
   - Check that your locale is set to your region. Use the command `locale` to view the current settings. Run `raspi-config` to set the locale for your region. Reboot for the changes to take affect.
2. Configure the network settings.
   - (Optional) Create a bridged interface. This can be handy if using other services such as qemu along with docker. Create the file `/etc/systemd/network/br0.netdev` using the editor of your choice. The file should contain the following settings:
   ```
   [NetDev]
   Name=br0
   Kind=bridge
   ```
   Next, create another file named `/etc/systemd/network/eth0.network` using the editor of your choice. The file should contain the following settings:
   ```
   [Match]
   Name=eth0

   [Network]
   Bridge=br0
   ```
   - (Optional) Add configuration for networkd to provide macvlan bridge for docker
   - Edit /etc/dhcpdcp.conf

3. Install docker using get-docker.sh script