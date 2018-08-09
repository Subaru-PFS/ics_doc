******
PFS services internal design rev.009 (pfs.ipmu.jp, pfspipe.ipmu.jp, internal to IPMU)
******

As of 2016/12/02

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

