Network assignment
======

Three VLANs are used, ``PFS-LAN`` (1096), ``IPMU-LAN``, and ``Global-LAN``. 
All network ports are configured as access port, no trunk port is used. 
Each switch has default VLAN for its main purpose, and some ports are 
configured as other VLAN(s) for management purpose. 

In the rack for services, five switches are used and configured as follows:

* ``PFS-LAN``: *c24a*, *f24b* (both all ``PFS-LAN``)
* ``PFS-LAN``/management: *ng72a* 
  (all ``PFS-LAN``, only management connection like iLO)
* ``IPMU-LAN``: *d5424a* 
  * ports 1-20 : ``IPMU-LAN``
  * ports 21-22 : trunk
  * port 23 : ``IPMU-LAM``
  * port 24 : ``PFS-LAN``
* ``Global-LAN``: *f24c* 
  * ports 1-12 : ``Global-LAN``
  * ports 13-22 : ``PFS-LAN``

LACP configurations
------

Following ports are configured as LACP groups.

* *c24a*/1: ports 1, 2 for *f24b* uplink connection
* *c24a*/2: ports 11, 12 for *dlb8-vm*
* *f24b*/1: ports 1, 2 as uplink to *c24a*

