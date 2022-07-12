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
* Make sure to run *-once scripts in [keel-openstack-kolla](https://github.com/niklasnorin/keel-openstack-kolla)/scrips

### Manila
* I had an issue creating shares, which was logged in Horizon Share User Messages as "allocate host: No storage could be allocated for this share request. Trying again with a different size or share type may succeed.".
* `manila service-list` showed no manila-share and `docker ps | grep manila-share` showed the service as unhealthy
* Diving into Kibana for programname:manila-share showed error "ERROR oslo_service.service Stderr: 'ovs-vsctl: no bridge named br-int\n'"
* FIX: On host `sudo ovs-vsctl add-br br-int`
