******
PFS services internal design rev.009 (pfs.ipmu.jp, pfspipe.ipmu.jp, internal to IPMU)
******

As of 2016/12/02

Network
******

Overall network diagram
======

All on PFS internal LAN

External interfaces

- To global: pfs.ipmu.jp (VM), pfspipe.ipmu.jp (VM)
- To IPMU internal: pfscalc (NFS and samba storage service)

Internal services

- Static lease based DHCP server (mgmt-dhcp) - testing on-site ICS
- RAID1 storages (pfsdisk; 3TB x3): Server storage, Virt disk store, 
  simulator storage / weekly disk health veryfy, 2 hotswap disk
- LDAP server (pfs.ipmu.jp)
- Most of physical server is just VM host server, except for pfsdisk (NFS 
  storage), pfscalc (IPMU internal services)

Connection
======

Ethernet connection for internal LAN is moving to managed L2 switches. 
For networks to global, keep using non-managed switch.

Servers

- PFS-LAN: Fujitsu SR-S324TC1 (24-4SFP; f24b .198)
- IPMU-LAN: Dell PC5424 (24-4SFP; d5424a 192.168.156.29)

Simulator rack

- Fujitsu SR-S324TC1 (24-4SFP; f24a .196) for master
- Cisco 2950 (48port 100M; c48a .195) for low speed hosts

Administration
******

Administrative resources
======

PFS internal wiki
  https://pfs.ipmu.jp/wiki/System : track on-going status
Munin status panels
  https://pfs.ipmu.jp/munin/
  http://himor.in/munin-cgi/ipmu.jp/ (backup)
libvirt VM panel
  Still TBD
mail lists
  Admin (pfs_pfs.ipmu.jp), munin (munin_pfs.ipmu.jp)

Service
******

External service management
======

Both pfs(srv) and pfspipe is on VM, rely on external services from VMs. 
Storage for server services are at pfsdisk:/server as follows.

/server/admin
  Administrative files including backup
/server/archive
  Archive of download or old data
/server/backup
  Backup of ldap, mysql, pgsql
/server/home
  To be mounted as /home
/server/httpd
  Web storage, http-* and https-* are mostly mounted directly as web
/server/services
  Storage for services: gitolite, jira, jira-pipe, mailman, munin, mysql, 
  postgresql
/server/storage
  To be open as https://pfs.ipmu.jp/*, like dd-images, ms-vl
/server/subversion
  Subversion repositories

Internal operation service management
======

mgmt-dhcp (.1)
  dnsmasq (DHCP)
mgmt (.7)
  munin for external view
pfsdisk (.3)
  RAID1 storage server (3TBx3, 2S): /server for external server data and home, 
  /virt for VM images, /simdata for data storage for simulators
landfill (.32)
  landfill services
db2 (.37)
  pgsql and mysql database service, and daily backup for every databases


