******
PFS services internal design (pfs.ipmu.jp, pfspipe.ipmu.jp, internal to IPMU)
******

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
For networks to IPMU and global, using (and will use) non-managed L2 switch.

Servers

- internal: Cisco 2960C (8+2SFP)
- IPMU-LAN: Dell PC5424

Simulator rack

- Fujitsu 24port 1G for master
- Cisco 2950 48port 100M for low speed hosts

Place of physical host
======

At wall side (server system) - from corner

- CAD PC
- pfsdisk: storage for services
- pfscalc: storage for internal use
- extVM2: host for external service VMs (port to global has no IP address)
- R410-1 (on desk): host for external service VMs

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
    network: r410-1 (external services), dl360-2 (Windows VM)
- service hosts: pfsdisk(.3), pfscalc(.4)

VM management
======

Virt disk storage on NFS

- /virt at pfsdisk (RAID1 3TB)
- /virt-win: at dl360-2 (RAID1 1TB) for Windows VMs used at host local

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
pfscalc3 (.6)
  RAID1 storage for /virt-win and Windows VM host
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
  Cs (8+2SFP; .190), CB2F (24+4SFPx2, FlexStack; .191, .192), SpS (24+4SFP; 
  .193), CB2F E-LAN (8+2SFP; .194), c48a (48; 100BASE; .195), 
  f24a (24; .196), p5424a (24; 192.168.156.29)
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
  Access 192.168.156.70:/data1, samba user/pass set at host by smbpasswd
Backup (6TB)
  "rsync" to 192.168.156.70:/data2, e.g. 
  ``rsync -a -delete --link-dest=<priv> <orig> <backup>``

Bots
======

Dropbox
  CIT dropbox running at jessie (by account ``atsushi.shimono``) and 
  syncing to ``192.168.156.70:/data1/cit-dropbox/Dropbox``.
  Check status via ``dropbox status``.


