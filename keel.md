# Keel home server

## Physical setup

### HDD
* 8x128GB SSDs in RAID5
* 16x600GB 10K SAS HDDs in RAID6
* Raids setup via iDRAC using physically connected screen and keyboard

### iDRAC
* Reset to default settings
* DHCP

## Host OS
* Ubuntu 20.04 LTS
* Installed via USB stick

### Manually installed

#### Partitioning storage
* sudo wipefs -a /dev/sda
* sudo wipefs -a /dev/sdb
* sudo pvcreate /dev/sda /dev/sdb
* sudo vgcreate ssd-cinder-volumes /dev/sdb
* sudo vgcreate hdd-cinder-volumes /dev/sda

#### Manual fan cron script
* Connect iDRAC Eth interface to local network and find IP
* sudo apt-get install ipmitool
* See [ipmi_fan_control.sh](ipmi_fan_control.sh)

#### OMSA - Dell EMC OpenManage
To get info out of PERC RAID cards.

* sudo apt-get install srvadmin-base srvadmin-storageservices srvadmin-omcommon
* sudo systemctl start dsm_sa_datamgrd && sudo systemctl start dsm_sa_eventmgrd

## OpenStack

### Setup
* Using Kolla Ansible
* See (private) [keel-openstack-kolla](https://github.com/niklasnorin/keel-openstack-kolla)
* Manually added default flavours

### One-time config

#### Add public SSH key
* openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey

#### Allow ping and SSH by default
* openstack security group rule create --proto icmp DEFAULT_SECURITY_GROUP_ID
* openstack security group rule create --proto tcp --dst-port 22 DEFAULT_SECURITY_GROUP_ID

## Glance examples

### Add e.g. Ubuntu
* Example from [here](https://computingforgeeks.com/adding-images-openstack-glance/)
* wget http://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```openstack image create \
    --container-format bare \
    --disk-format qcow2 \
    --file focal-server-cloudimg-amd64.img \
    Ubuntu-20.04```
