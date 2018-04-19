PFS VM guest MAC address block assignments
******

This document is applicable to MAC address assignment for VM guests to 
subsystems at production. Individual assignments to VM guests are registered 
to ics_dnsmasq repository. 

Backgrounds
======

- Prevent using the same MAC address by multiple VM guests both within PFS ICS and with ones in Subaru summit network.
- To make ease on checking conflict at registration on ics_dnsmasq repository, assign a block to each (group) of subsystem which owns a configuration file in ics_dnsmasq.
- Assign a block by not smaller than the last hex digit, from 06:50:00:A5:0[01]:XX provided from Subaru.

MAC address assignments
======

Before defining individual VM guest in ics_dnsmasq, check all of corresponding 
configuration file (for pairs of MAC address and hostname) in the repository, 
including non-merged branches. 

- **06:50:00:A5:00:\[0-7\]X**: PFS ICS services

  - \*-ics, \*-infra, support tools

- **06:50:00:A5:00:8X**: SpS/ENU (ENU control host)
- **06:50:00:A5:00:9X**: SpS/SCR

  - control and monitoring of SCR and support infrastructure

- **06:50:00:A5:00:\[ab\]X**: PFI
- **06:50:00:A5:00:cX**: MCS
- **06:50:00:A5:00:dX**: Landfill
- **06:50:00:A5:00:fX**: (kept as spare)
- **06:50:00:A5:01:XX**: AIT and development site specific tools

  - Assuming not to be delivered to the production
  - Recommended to follow conventions for '00'

