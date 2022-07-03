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
* 

#### OMSA - Dell EMC OpenManage
To get info out of PERC RAID cards.

* sudo apt-get install srvadmin-base srvadmin-storageservices srvadmin-omcommon
* sudo systemctl start dsm_sa_datamgrd && sudo systemctl start dsm_sa_eventmgrd

## OpenStack

### Setup
* Using Kolla Ansible
* 
