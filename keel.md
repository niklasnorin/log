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

### Troublehooting "br-int does not exist" errors
* Sanity check to see if this is an issue:
  * When everything works, then `sudo ovs-vsctl show` on the host should spit out a ton of info like:
```
keel@keel:~$ sudo ovs-vsctl show
42cec4e6-b7f6-4292-bb7d-58981130a503
    Manager "ptcp:6640:127.0.0.1"
    Bridge br-tun
        Controller "tcp:127.0.0.1:6633"
        fail_mode: secure
        datapath_type: system
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
        Port br-tun
            Interface br-tun
                type: internal
    Bridge br-int
        Controller "tcp:127.0.0.1:6633"
        fail_mode: secure
        ...
        ... cut for bravity
        ...

    Bridge br-ex
            Controller "tcp:127.0.0.1:6633"
            fail_mode: secure
            datapath_type: system
            Port br-ex
                Interface br-ex
                    type: internal
            Port phy-br-ex
                Interface phy-br-ex
                    type: patch
                    options: {peer=int-br-ex}
            Port eno2
                Interface eno2

```


* Check that openvswitch related containers are healthy
  * `sudo docker ps | grep openvswitch`
* Check logs
  * neutron_openvswitch_agent: kibana search `programname:"neutron-openvswitch-agent"`
  * openvswitch_db: `sudo tail -100 /var/log/kolla/openvswitch/ovs-vswitchd.log`
  * openvswitch_db: `sudo tail -100 /var/log/kolla/openvswitch/ovsdb-server.log`
* I had to 
  * stop all the above docker containers
  * find rogue ovs related processes using `ps aux | grep ovs` and `ps aux | grep ovsdb`
  * kill them using `kill -15 PID`
  * remove any .pid files `sudo rm /var/run/openvswitch/...`
  * then start containers in following order
    * openvswitch_db
    * openvswitch_vswitchd (it's log will spew out a "cannot connect" error until neutron_openvswitch_agent started)
    * neutron_openvswitch_agent

### Creating manila share
* Create share: `manila create NFS 1 --name demo-share2 --share-network demo-share-network1 --share-type default_share_type --public`
* Whitelist access from home network: `manila access-allow demo-share2 ip 192.168.1.0/24`
* Mount on OSX CLI: `sudo mount -o vers=4,resvport -t nfs 192.168.1.163:/shares/share-51e5f995-14b4-409b-ba89-634be72b098c ./mount_test`
* Mount on Ubuntu:
  * Install NFS support `sudo apt install nfs-common`
  * Mount: `sudo mount -v 192.168.1.163:/shares/share-51e5f995-14b4-409b-ba89-634be72b098c ./test_folder`  
* Only tested public (no user/pass) share so far 
