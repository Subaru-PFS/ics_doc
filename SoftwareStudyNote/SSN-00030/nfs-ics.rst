Shared storage service for PFS ICS production
******

This note covers configurations on FC/iSCSI storage device and NFS services. 

* `Storage device configuration`_
* `NFS service configuration`_

Storage device configuration
======

Overview
------

PFS ICS production infrastructure has one set of FC/iSCSI storage device, 
which is consisted with 2 units of 12 6TB HDD drives. 
Storage is configured as two RAID6 disk groups with 2 global hot spare disk, 
and is providing 108TB total size (2 groups of 54TB RAID6 pool). 
Host interface has 8 data ports and 2 management ethernet ports, 
and data ports are configured as 4 8Gbps FC and 4 1Gbps iSCSI channels.
4 iSCSI channels are connectable in bonded using device-multipath configuration 
at each host (no LACP required). 

Access
------

All interfaces are named as 'issAB-infra' with A for port category ('d' for 
data and 'a' for management), B for port ID in each category. 
iSCSI ports are configured as static IP assignment, management ports are 
configured as DHCP. 

LUNs
----

Following LUNs are defined for each target area, names are label in storage 
device configuration. 

- VMGuest: 5TB, for VM guest images with its backup and archive.
- ProdData: 70TB, for production (FITS) data.

  - This LUN could be divided into 5, like 4 LUNs for each IR camera storage 
    and 1 LUN for all rest device. 

- OperData: 5TB, for system operation mounted from VM guests, 
  such as 'home', operational data including logs or running outputs from 
  actors, service data storage except for operational PostgreSQL, and 
  temporary shared data (for e.g. on-site quick analysis).
- Pgsql: 10TB, for PostgreSQL database storage, and this LUN is assumed to 
  be mounted directly from database server via iSCSI. 
- Backup: 18TB, for backup and logs, such as differential backup of 'OperData' 
  LUN, or raw dump of rsyslog output. 

  - On-site backup for entire PostgreSQL database (both binary or SQL dump) 
    is not planned, but partial backup of important databases for instrument 
    operation (like configuration) could be planned. 
  - Incremental backup using hard link for unchanged files would save data 
    volume required for periodical backup, but we may need to consider 
    decimating old ones after ~years. 

Memo
----

- Onsite and online database full backup (full dump or binary copy) is not 
  in target. Replication to external offsite place need to be planned, such as 
  to Hilo, both far site backup and remote reference. 
- The largest target of 'ProdData' is one from IR detectors, which has data 
  volume of 32MBx4 per ~5.5sec in raw data (no compression), and 15TB for one 
  observation run with 12h exposure time and 2 weeks. 

  - Roughly placed requirement on keeping production data in onsite storage 
    is to keep around a half year, with having important or reference data to 
    be used for engineering or maintenance. 
    This storage is not a final destination or indefinite archive, and this 
    period is just a reference value. 
  - In the real operation, some compression is planned to be applied, like 
    typical 0.3-0.4 by Rice or double incremental differential with Rice. 

- Since backup in this storage is on the same one for the operation, it is 
  better to consider of an external backup for important data. 


NFS service configuration
======

Overview
------

One physical host is planned to mount storage via FC (4Gbps or 8Gbps), and 
to provide shared storage via NFS (v4 in target, v3 for now). Depends on its 
performance or resource management, this host could be a VM host and some 
NFS server might be operated as VM guest mounting block device via VM layer. 

Access
------

To make client configuration stable even when infrastructure decides to devide 
NFS server using VM layer, a dedicated hostname is to be assigned to each 
block device as following, and it is recommended to mount via a pair of 
hostname and mount point. 

- nfs-ics: Alias for physical host itself
- nfsv-ics: Alias for 'VMGuest' area
- nfsp-ics: Alias for 'ProdData' area
- nfso-ics: Alias for 'OperData' area
- nfsb-ics: Alias for 'Backup' area

Memo
----

- Physical host of NFS service will have connection via FC to storage and 
  bonded ethernet to NFS client hosts, data flow to two (categories of) targets 
  will flow through different path and will not cause degradation from flow 
  duplication at storage device connection point. 

  - Note, in case of trouble, like FC card broken, we need to switch FC 
    connection to iSCSI, and it will make theoretical maximum data flow into 
    half. 
  - Requirement to storage device performance is 3-4Gbps in sequential block 
    read/write, connection of both FC and bonded ethernet for NFS could be ok 
    with 4Gbps although we have options to upgrade to 8Gbps FC or 10Gbps 
    metal ethernet by hardware upgrade. 

- One LUN is assigned to one storage mount point exposed to NFS. 

  - This enable us to divide NFS server one mount point by one if we have some 
    performance issue like both data flow and node operation at host. 
  - Each LUN will be shown up as a block device to a physical host, and can 
    be exposed to VM guest (with directsync). 

