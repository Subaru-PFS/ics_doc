Services overview
======

ToC

* `Installation and configuration`_
* `Public servers`_
* `Backend services`_
* `Service or API providers`_
* `Current status on ansible`_ 


Installation and configuration
------

Installation of operating system is by preseeded setup CD-R, which will build 
base system as similar configuration (not the same, could depends on resource 
configuration like memory or disk image size), and configuration of system 
is planned to be fully performed using ansible for version tracked and easily 
replayable system configuration (refer `Current status on ansible`_ section 
for production usage). 

Except for service configuration (mostly ones in /etc, which are now moving 
into version tracked repository), data store for service operation are mostly 
provided through NFS or disk image at ``/virt`` (limited for temporal storage 
or data store with periodic backup), which makes replacement of VM guest image 
to be easy. 

Public servers
------

Three public servers which has a global address are running. 
All servers are under ipmu.jp domain. 

* ``pfs``: main server

  * web site, both locally working services and via reverse proxy
  * mail service (incoming, maillist, and mailbox)

* ``pfspipe``: dedicated to software development

  * web site, mostly via reverse proxy
  * mail service (maillist)

* ``pfsarch``: archive storage server

  * web site, for data input and output

For future integration, ``pfspipe`` could be integrated into ``pfs`` server 
as virtual host. 
``pfsarch`` was devided into single VM guest on data flow performance point of 
view (data are on iSCSI storage but not on main service storage, so data flow 
is totally separated from ``pfs`` or ``pfspipe``). 

Backend services
------

Backend and infrastructural services are provided from dedicated VM guest 
with alias hostname for service. Most of services are configured by ansible. 
Following hostnames are defined for backend serviecs, and clients are 
recommended to use service hostname but not its server hostname: 

* ``ldap``: LDAP account server
* ``smarthost``: PFS-LAN internal mail smarthost
* ``ntp``: ntp server
* ``rsyslog``: rsyslog receive server
* ``mysql``: mysql server
* ``pgsql``: postgresql server
* ``influxdb``: influxdb server

Note:

* NFS storage configuration for VM host (``pfsdisk``) shall be by IP 
  address but not canonical name, since they need to run 
  without DHCP/DNS server. 
* Canonical hostnmaes are registered in dnsmasq repository.

Service or API providers
------

Services and APIs provided through web reverse proxy are hosted by backend 
servers. Configurations of reverse proxy are on version tracked repository, 
and service configurations and reverse proxy configurations shall be updated 
in consistent. 

Current status on ansible
------

Following hosts are fully configured by ansible playbook (real ones are 
configured by ansible without any manual operation): 

* storage hosts (*pd*, *pc*, *sd*)
* landfill services (only for template)
* ``rsyslog``

Following hosts are fully configurable by ansible playbook, but need some 
manual operation: 

* (TBC)

Following hosts are currently under development and/or confirmation:

* external servers (``pfs``, ``pfspipe``, ``pfsarch``)
* database servers (``mysql``, ``pgsql``)

Following hosts are not planned yet: 

* JIRA

