# Keel home server

## Physical setup

### HDD
* 8x128GB SSDs in RAID5
* 16x600GB 10K SAS HDDs in RAID6


### iDRAC
* Reset to default settings
* DHCP

## Host OS config

### Manually installed

#### OMSA - Dell EMC OpenManage
To get info out of PERC RAID cards.

* sudo apt-get install srvadmin-base srvadmin-storageservices srvadmin-omcommon
* sudo systemctl start dsm_sa_datamgrd && sudo systemctl start dsm_sa_eventmgrd
