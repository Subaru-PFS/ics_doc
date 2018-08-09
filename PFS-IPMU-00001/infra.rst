Infrastructure overview (storage, VM host)
======

ToC

* `Storage service configuration`_
* `VM host configuration`_

Storage service configuration
------

Two categories of storage services are provided, one is actively used service 
backend, and another is archive and/or backup service backend. 

For actively used service backend, storage is exposed through VM guest which 
mounts physical drive through virtio. 
Disk drives are placed in one SAS expander box and controlled by RAID card at 
host computer as RAID1 with hot spare. On hardware issue, it is possible to 
move disk drives into another host computer to continue services. 
To survive UPS failure, both SAS expander box and host computer shall be 
connected to two separated UPS as redundant power source. 
Four RAID1 are mounted and exposed by three VM guests: 

* *pd* (``pfsdisk``): ``/virt`` and ``/server``
* *pc* (``pfscalc``): ``/data1``
* *sd* (``simdata``): ``/simdata``


For archive service backend, we have one iSCSI connected RAID6 storage server 
and two VM guests to host storage via iSCSI. 
The storage server is not easily replacable, and need to consult with vender 
on hardware issue. Disk drives are configured as RAID6 with hot spare. 
Two VM guests are for: 

* PFS development data archive (``pfsarch``)

  * Storage is exposed to outside via NFS 
    (to be used by ``pfs.ipmu.jp`` server) 
    and httpd. 
  * This archive is for archive of raw data from real hardware or simulation. 

* PFS internal backup (``backup``)

  * Storage is not exposed to outside backup system, not to be edited nor 
    modified by any system running inside PFS-LAN, including malwares. 
  * Backup system mounts service storage via NFS, and perform differential 
    backup once per week (configured at weekend). 
  * Some exclusions are defined for backup of server storage, like binary data 
    store (database). 

======
Directory assignments
======

------
/server
------

/server/admin
  Administrative files
/server/archive
  Archive of downloaded package or old data
/server/backup
  Backup of service data (like ldap, mysql, pgsql)
/server/home
  To be mounted as /home
/server/httpd
  Web storage, http-* and https-* are mostly mounted directly as web
/server/services
  Service data storage, each service shall have its own sub-directory
/server/storage
  Internal use data store, to be open at internal website

VM host configuration
------

Except for one used as storage service host computer, 
configurations of all VM host computers shall be consistent and replacable 
without special care nor handling. 

Hardware configuration of VM host computers are as follows.

* Requirement

  * RAID1 host system storage (no hot spare assumed)
  * minumum 2GB/HTcore memory with ECC enabled
  * two network bridge connection, one to ``PFS-LAN`` with IP address as 
    ``br0``, another to ``IPMU-LAN`` without IP address as ``br2``

* Optional

  * one additional network bridge connection to ``Global-LAN`` without IP 
    address as ``br1``

System configuration are all performed using ansible playbook, except for 
network bridge configuration (``br0`` is configured but not br1/2, as for now). 

* System account to run libvirt etc.
* libvirt environment including server certificate files for libvirt PKI
* fixed IP address configuration for ``br0`` (``PFS-LAN``)
* Exporters for prometheus monitoring (node, RAID)
* Basic system configuration (ntp, smarthost, rsyslog)

Configurations shall follow above requirements to enable hot migration of VM 
guests among available VM host servers, including assignment of br0/1/2. 
In PFS server system, only a few VM guests have connection to ``Global-LAN``, 
so ``br1`` network connection is not a requirement over system-wide (we can run 
these VM guests with small number of physical hosts). 

