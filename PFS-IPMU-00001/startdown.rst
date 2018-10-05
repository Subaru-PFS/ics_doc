Service cold start and full shutdown procedures
------

This memo shows procedure to start the all infrastructres from cold and to shutdown the all, with points 
in consideration or taken cared of.

Cold start
======

Start backends first, end point to external, and monitoring systems. 

* UPS and network switches will come up automatically after power back

  * Wait for UPS charged up enough (at least 30 %). Reset UPS if needed.
  * Check the network switches are up by looking the status LEDs

* Power on storage server

  * iSCSI server needs to be started with JBOD, and controller follows
  * Just power on SAS storage box

* Power on VM host for NFS servers (*dlb8-vm*)
* Start VM guests of NFS servers
  (``pfsdisk``, ``pfscalc``, ``simdata``; no order)
  * *pd2*, *pc2*, *sd2*
* Start core management service (*mgmt3*, including DHCP, LDAP)
  * The host is *dlb5-vm*
* Start backend services (no order)

  * ``Database`` : * *db3*
  * ``rsyslog`` : *rsyslog*
  * Elasticsearch (log target of rsyslog) :*elasticsearch*
  * ``backup`` : *backup*

* Start external servers (``pfs``, ``pfspipe``, ``pfsarch``)
  * *pfssrv*, *pfspipe2*, *pfsarch*
* Start service VM guests

  * ``jirapipe`` : *jira-pipe-7.7*

    * Start jira server instance manually via systemd
	ssh pfs@jirapipe-77
	sudo systemctl start jira

  * *jupyter-spt*, *jupyter1*

* Start monitoring services (``prometheus``, ``influxdb``)
  * *prometheus*, *influxdb*, *prometheus-snmp*
* Start internal services
  * *landfill*, *stretch*, *jessie-main*
  * ``Windows`` : *WinP1*, *WinVSdev* and CAD
* Check the above VM guests running
* Check internal services up by accessing them.

Full shutdown
======

Shutdown end point to external to prevent unwanted connection first, and 
shutdown backends. 
See the section `Start & shutdown VM guest with virsh`_ for the virsh command to shutdown the VM guests.
Refer the mail (subject: \"List of virt VMs\") sent to admin weekly for VM guests running on individual hosts.

* Shutdown monitoring services (e.g. ``prometheus``, ``influxdb``)
  * *prometheus*, *influxdb*, *prometheus-snmp*
* Shutdown internal servicse (like working shell host, simulator)
  * *lf??*, *landfill*, *nfs??*, *stretch*, *jessie-main*
* Shutdown external servers (``pfs``, ``pfspipe``, ``pfsarch``)
  * *pfssrv*, *pfspipe2*, *pfsarch*
* Shutdown iSCSI storage for pfsarch via web admin panel
  * On browser type IP of pfsarch server (pas-srv, see `dnsmasq<https://github.com/Subaru-PFS/ics_dnsmasq/blob/master/hosts-ipmu/srv.conf>`_), then shutdown
  * See PO internal wiki for sign-in information
* Shutdown service VM guests (``jupyter``, ``jirapipe``, ``windows``)
  * *jupyter-spt*, *jupyter1*, *jira-pipe-7.7*
  * For Windows (*WinP1*, *WinVSdev*, and CAD), connect via e.g. Remode Desktop to apply updates before shutdown.
* Shutdown backend services
  * *db2*, *db3*, *backup*, *mgmt3*, *elasticsearch*, *rsyslog*
* Shutdown VM host server (except for one running core management guest)
* Shutdown core management guest (DHCP, LDAP) and its host (*dlb5-vm*)
  * *mgmt3*
* Shutdown NFS server VM guest
 * *pc2*, *sd2* (VM guests running on the same VM host as one for NFS server)
 * *pd2*
* Shutdown NFS server VM host (*dlb8-vm*)

Start & shutdown VM guest with virsh
=====

* To start ``vm guest`` on ``vm host``,

	virsh -c qemu+tls://``vm host``/system start ``vm guest``

* To shutdown ``vm guest`` on ``vm host``,

	virsh -c qemu+tls://``vm host``/system shutdown ``vm guest``

* If ``vm guest`` won't shutdown, use

	virsh -c qemu+tls://``vm host``/system destroy ``vm guest``

* To list the VM guests running on ``vm host``

	virsh -c qemu+tls://``vm guest``/system list --all
