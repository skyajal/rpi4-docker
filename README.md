# Pine64 docker compose
This is a list of docker compose entries tested on the pine64 for arm compatibility and development.
Please note that this is only information that has been gathered as part of my testing. Your results may vary.

## Docker compose file example
This example uses version 3 of the compose file format. Additional settings
can be found here: https://docs.docker.com/compose/compose-file

## Docker macvlan
Use the following to create a macvlan for the stack to use. Replace the parent
device and networking information. Below is an example to use IP addresses from
a network range not used by dhcp. I opted not to create the network settings
in the compose file. Additional information: https://docs.docker.com/network/macvlan/

Note: The option -o "com.docker.network.macvlan.name"="macvlan" is optional.

docker network create -d macvlan -o parent=eth0 \
-o "com.docker.network.macvlan.name"="macvlan" \
--subnet 192.168.0.0/24 --gateway 192.168.0.1 \
--ip-range 172.16.0.88/30 macvlan

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
after a reboot. Please see https://gist.github.com/timcharper/d547fbe13bdd859f4836bfb02197e295 and https://gist.github.com/jooter/8f95555021d2a4e7cb6241368473ed55 for example workarounds.

## Docker bridging
Use the following to create a bridge for the stack to use. The default bridge
network in docker does not allow containers to connect each other via container
names used as dns hostnames. Therefore, it is recommended to first create a user
defined bridge network and attach the containers to that network. This must also
be done prior to using docker-compose. I opted not to create the network settings
in the compose file. Change the bridge name to whatever you want along with the
subnet and gateway IP. Keep armnet the same or replace all instances
with the name you want to use. See The example below:

docker network create --driver=bridge --subnet=192.168.10.0/24 \
--gateway=192.168.10.1 \
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

## Containers used in the example
Portainer
https://portainer.readthedocs.io/en/stable/deployment.html

Adguard Home
https://hub.docker.com/r/adguard/adguardhome
https://github.com/AdguardTeam/AdGuardHome
