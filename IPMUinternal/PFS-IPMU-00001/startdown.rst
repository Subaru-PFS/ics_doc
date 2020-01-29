Service cold start and full shutdown procedures
------

The startup and shutdown procedure of all infrastructures is described below.
The following notation is used in this memo unless otherwise noted:

* **VM host name** (e.g., **dlb8-vm**)

* ``VM guest name``  (e.g., ``pd2``)

* *Service name* (e.g., *pfsdisk*)

Note: before and after carry out the procedure, send an announcement e-mail to the PFS tech team.

Cold start
======

Start back-ends first, then end-points to external, and monitoring systems.
See `Start & shutdown VM guest with virsh` section for the `virsh` command to startup the VM guests.

* UPS and network switches will come up automatically after power back

  * Wait for UPS charged up enough (at least 30 %). Reset UPS if needed.
  * Check the network switches are up by looking the status LEDs

* Power on storage server

  * iSCSI server needs to be started with JBOD, and controller follows

    * Turn on the switches in the bottom first, and then those in the top

  * Just power on SAS storage box

    * Press switch for a few second (TBC)

* Power on VM host for NFS servers (**dlb8-vm**)

* Start VM guests of NFS servers
  (*pfsdisk*, *pfscalc*, *simdata*; no order)

  * ``pd2``, ``pc2``, ``sd2`` on **dlb8-vm**

* Power on other host servers (**dlb3-vm**, **dlb5-vm**, **dlc2-vm**, ...)

* Start core management service (``mgmt3``, including DHCP, LDAP)

  * ``mgmt3`` on **dlb5-vm**

* Start backend services (no order)

  * *database* : ``db3`` on **dlb5-vm**
  * *rsyslog* : ``rsyslog`` on **dlb5-vm**
  * *elasticsearch* : ``elasticsearch`` on **dlb3-vm** (log target of ``rsyslog``)
  * *pfsarch* : ``pfsarch`` on **dlc2-vm** (one area exported from this NFS server is mounted at ``pfssrv`` for archive storage export point)

* Start backup service

  * *backup* : ``backup`` on **dlb5-vm**

* Start external servers

  * *pfs* : ``pfssrv`` on **dlc2-vm**
  * *pfspipe* : ``pfspipe2`` on **dlc2-vm**

* Start service VM guests

  * *jirapipe* : ``jira-pipe-8.3`` on **dlb3-vm**

    * Start jira server instance manually via systemd

	ssh pfs@jirapipe-83

	sudo systemctl start jira

  check on browser whether JIRA is starting up

  * *jupyter-spt* : ``jupyter-spt`` on **dlc3-vm**
  * *jupyter1* : ``jupyter1`` on **dlb4-vm**
  * *dbsim2* : ``dbsim2-spt`` on **rcc1-vm**

* Start monitoring services

  * *prometheus* : ``prometheus`` on **dlb5-vm**
  * *influxdb* : ``influxdb`` on **dlb3-vm**
  * *prometheus-snmp* : ``prometheus-snmp`` on **dlb5-vm**

* Start internal services

  * ``landfill``, ``stretch``, and ``jessie-main`` on **dlb3-vm**
  * *Windows* : ``WinP1``, ``WinVSdev`` (and CAD?) on **dlb4-vm**

* Check the above VM guests running
* Check internal services up by accessing them.

Full shutdown
======

Shutdown the end-point to the external services to prevent unwanted connection first, and 
then back-end services. 
See `Start & shutdown VM guest with virsh` section for the `virsh` command to shutdown the VM guests.
Refer the mail (subject: \"List of virt VMs\") sent to admin weekly for VM guests running on individual hosts just in case.

* Shutdown monitoring services (e.g. *prometheus*, *influxdb*) not to get unwanted (or obvious) warnings from hosts shut down.

  * *prometheus* : ``prometheus`` on **dlb5-vm**
  * *influxdb* : ``influxdb`` on **dlb3-vm**
  * *prometheus-snmp* : ``prometheus-snmp`` on **dlb5-vm**

* Shutdown internal services (like working shell host, simulator)

  * ``jessie-main``, ``landfill``, and ``stretch`` on **dlb3-vm**
  * ``nfs??`` on **dlb2-vm** (disused)
  * ``lf??`` on **dlb7-vm** (disused)

* Shutdown external servers

  * *pfs* : ``pfssrv`` on **dlc2-vm**
  * *pfspipe* : ``pfspipe2`` on **dlc2-vm**
  * *pfsarch* : ``pfsarch`` on **dlc2-vm**

* Shutdown backup service

  * *backup* : ``backup`` on **dlb5-vm**

* Shutdown iSCSI storage for pfsarch via web admin panel

  * On browser type IP of *pfsarch* server (`pas-srv`, see `dnsmasq<https://github.com/Subaru-PFS/ics_dnsmasq/blob/master/hosts-ipmu/srv.conf>`_), then shutdown
  * See PO internal wiki page for sign-in information

* Shutdown service VM guests (*jupyter*, *jirapipe*, *Windows*, etc.)

  * *jupyter-spt* : ``jupyter-spt`` on **dlc3-vm**
  * *jupyter1* : ``jupyter1`` on **dlb4-vm**
  * *dbsim2* : ``dbsim2-spt`` on **rcc1-vm**
  * *jirapipe* : ``jira-pipe-8.3`` on **dlb3-vm**
  * *Windows* : ``WinP1``, ``WinVSdev`` and CAD? on **dlb4-vm** 
    * Note: connect via e.g. Remote Desktop to apply updates before shutdown.

* Shutdown back-end services

  * *database* : ``db3`` on **dlb5-vm**
  * *management* : ``mgmt3`` on **dlb5-vm**
  * *elasticsearch* : ``elasticsearch`` on **dlb3-vm**
  * *rsyslog* : ``rsyslog`` on **dlb5-vm**

* Shutdown VM host servers (except for **dlb5-vm** running core management guest)
* Shutdown core management guest (DHCP, LDAP)
  * ``mgmt3`` on **dlb5-vm**
  * Shutdown the host server (**dlb5-vm**)
* Shutdown the NFS server VM guests

  * ``pc2``, ``sd2`` on **dlb8-vm** (VM guests running on the same VM host as one for NFS server)
  * ``pd2`` on **dlb8-vm**

* Shutdown the NFS server VM host (**dlb8-vm**)


Start & shutdown VM guest with virsh
=====

* To start ``vm guest`` on **vm host**,

	virsh -c qemu+tls://**vm host**/system start ``vm guest``  (from other host) 

  sudo virsh start ``vm guest`` (on **vm host**)

* To shutdown ``vm guest`` on **vm host**,

	virsh -c qemu+tls://**vm host**/system shutdown ``vm guest``  (from other host) 

  sudo virsh shutdown ``vm guest`` (on **vm host**)

* If ``vm guest`` won't shutdown (most likely when a trouble happens), use

	virsh -c qemu+tls://**vm host**/system destroy ``vm guest``  (from other host) 

  sudo virsh destroy ``vm guest`` (on **vm host**)

* To list the VM guests running on **vm host**,

	virsh -c qemu+tls://**vm host**/system list --all  (from other host) 

  sudo virsh list --all (on **vm host**)

