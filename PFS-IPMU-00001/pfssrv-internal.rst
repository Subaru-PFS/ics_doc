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

Place of physical host
======

Server system at wall side, on floor

- pfsdisk: storage for services
- pfscalc: storage for internal use
- extVM2: host for external service VMs (port to global has no IP address)

Server system at wall side, on work desk

- r410-2: VM host for external services
- dl360-1, dl360-4: Windows host (physical)

At work desk (simulator services)

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

Hardware computing resources
======

Base configurations at https://pfs.ipmu.jp/wiki/System/install/linux-base

Bridges for physical ethernet ports (to make virt live migration possible)

- br0 : PFS-LAN
- br1 : external
- br2 : IPMU internal

Hosts

- See list at https://pfs.ipmu.jp/wiki/System/hardware
- external server (br0 to PFS-LAN, br1 to internal) run only VMs requires 
  global address: extvm2(.5)
- internal server (br0 to PFS-LAN, br2 to IPMU) run only VMs requires IPMU 
  network: r410-2 (external services)
- service hosts: pfsdisk(.3), pfscalc(.4)

VM management
======

Virt disk storage on NFS

- /virt at pfsdisk (RAID1 3TB)
- Local storage only for host operation
- VM operation via virsh interface, remote monitoring via libvirt feature 
  under testing
- VM hosts could be easily replaced, no configuration difference among hosts, 
  except for network bridge (existence of bridges to IPMU or global)

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

System startup procedure
======

- Power on (at panel) and wait for startup of network switch
- Power on pfsdisk, system health check (on KVM)
- Power on extvm2, system health check (on KVM)
- Power on mgmt-dhcp, pfssrv, pfspipe on extvm2, and check services (apache, 
  mailman) on pfssrv
- Restart ntp service on pfsdisk, extvm2
- Check LDAP loading on pfsdisk (LDAP server at pfssrv)
- Power on pfscalc, check eth ar up
- Power on VM host servers (no order)
- Bootup service VMs. Mostly no order, that services rely on DB will resume 
  their connection on db startup

PFS instrument simulator
======

iSCSI storage server (.170-.179)
  About 100TB RAID6 iSCSI storage, connected by iSCSI device multipath 
  (.170-.177), and server admin IF (.178, .179)
Axis PTX surveillance camera (.180)
  Testbed for SpS/SCR
Cisco switches
  CB2F (24+4SFPx2, FlexStack; .191, .192),
  SpS (24+4SFP; .193)
KVM
  simulator (10.100.200.203, 192.168.156.33), 
  server (10.100.200.212, 192.168.156.32)
PDU
  simulator (.204), server (.211)

Service for IPMU internal
******

Shared storage
======

Shared storage service is provided as 192.168.156.70 (10.100.200.4) in RAID1 
4TB and 6TB.

NFS, samba (4TB)
  Access 192.168.156.70:/data1, samba user/pass set at pfscalc by smbpasswd. 
  All data are backed up to /data2 per weekly.
Backup (6TB)
  "rsync" to 192.168.156.70:/data2, e.g. 
  ``rsync -a -delete --link-dest=<priv> <orig> <backup>``

Bots
======

Dropbox
  CIT dropbox running at jessie (by account ``atsushi.shimono``) and 
  syncing to ``192.168.156.70:/data1/cit-dropbox/Dropbox``.
  Check status via ``dropbox status``.


