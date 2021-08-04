# Raspberry Pi 4 - Docker Setup with Raspbian OS 64 bit

Below is the steps that were used to setup docker on Raspbian OS 64 bit. Detailed information will be added as time permits.

1. Install Raspbian OS 64 bit
2. Configure network
    a. (Optional) Create bridged interface
    b. Edit /etc/dhcpdcp.conf
    c. (Optional) Add configuration for networkd to provide macvlan bridge for docker
3. Install docker using get-docker.sh script
