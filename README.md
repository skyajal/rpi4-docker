# Raspberry Pi 4 - Docker Setup with Raspbian OS 64 bit

Below is the steps that were used to setup docker on Raspbian OS 64 bit. Detailed information will be added as time permits.

1. Install Raspbian OS 64 bit
2. Configure network
   - (Optional) Create bridged interface
   - Edit /etc/dhcpdcp.conf
   - (Optional) Add configuration for networkd to provide macvlan bridge for docker
3. Install docker using get-docker.sh script

1. Install Raspbian OS 64 bit. Once this step has been completed, here are a few things to check before moving on to the next step:
   - Check that the timezone is set correctly