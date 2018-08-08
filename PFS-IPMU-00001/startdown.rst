Service cold start and full shutdown procedures
------

This memo shows ways to start all from cold and to shutdown all, with points 
in consideration or taken cared of.

Cold start
======

Start backends first, end point to external, and monitoring systems. 

* UPS and network switches will come up automatically after power back

  * Check these are up by checking status LEDs

* Power on storage server

  * iSCSI server need to be started from JBOD, and controller follows
  * Just power on SAS storage box

* Power on VM host for NFS servers (*dlb8-vm*)
* Start VM guests of NFS servers
  (``pfsdisk``, ``pfscalc``, ``simdata``; no order)
* Start core management host (*mgmt-dhcp*, including DHCP, LDAP)
* Start backend services

  * DB
  * ``rsyslog``
  * Elasticsearch (log target of rsyslog)

* Start service VM guests

  * jirapipe

    * Start jira server instance manually via systemd

  * landfill

* Start external servers (``pfs``, ``pfspipe``, ``pfsarch``)
* Start monitoring services (``prometheus``, ``influxdb``)
* Start internal services

Full shutdown
======

Shutdown end point to external to prevent unwanted connection first, and 
shutdown backends. 

* Shutdown monitoring services (``prometheus``, ``influxdb``)
* Shutdown internal servicse (like working shell host, simulator)
* Shutdown external servers (``pfs``, ``pfspipe``, ``pfsarch``)
* Shutdown iSCSI storage for pfsarch (via web admin panel)
* Shutdown service VM guests
* Shutdown backend services
* Shutdown VM host server (except for one running core management guest)
* Shutdown core management guest (DHCP, LDAP) and its host
* Shutdown NFS server VM guest
* Shutdown NFS server VM host

