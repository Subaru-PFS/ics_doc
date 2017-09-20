PFS ICS hostname Conventions
******

- This document is applicable to 'hostname's used in PFS ICS
- Follow RFC #2119 for requirement levels

**TOC**

- `Naming hostname`_
- `Keywords`_

Naming hostname
======

- PFS ICS hostnames SHALL be named as `\<feature\>-\<subsystem\>` format, 
  like `bee-b1` for BEE used in BCU1. 
- Both `\<feature\>` and `\<subsystem\>` SHALL be named in some charactors of 
  lower case alphabet with following numerical charactors for multiple 
  instances. 

  - Following numerical charactors are just to indicate multiple instances 
    like `sm1`, `sm2`, `sm3`, `sm4` for four spectrograph module instances, 
    but not added for single instance ones like `pfi`.
  - Following numerical charactors MAY be 2 digits in `\<feature\>` for 
    largely multiplated devices, but SHALL be as short as possible. 

- Alphabet part of `\<subsystem\>` SHALL be selected from listed in `Keywords`_ 
  section, which are picked up from a list of PFS acronyms of subsystems.
- Alphabet part of `\<feature\>` is RECOMMENDED to be selected from acronyms 
  listed in the PFS product tree.

Keywords
======

This section lists keywords to be used as `\<subsystem\>` part, and following 
`\[1-4\]` part in definitions are possible range of multiple instances. 
(If something missing, one SHALL be added to this list before using.)

pfi
  Prime Focus Instrument, including control software module at CB2F.
sm\[1-4\]
  Spectrograph Module control, usually called as ENU part.
b\[1-4\]
  BCU (Blue Camera Unit) control, shorten for already existing implementations.
r\[1-4\]
  RCU (Red Camera Unit) control, shorten for already existing implementations.
n\[1-4\]
  RCU (NIR Camera Unit) control, shorten for already existing implementations.
scr
  Spectrograph Clean Room control, also called as SpS infrastructure.
mcs
  Metrology Camera System, mounted at Cs focus.
vm
  VM host cluster at CB2F, use `scr` for ones at TUE-IR.
ics
  ICS operational infrastructure, including MHS, DB server, support services.
infra
  ICS hardware infrastructure, including PDU, KVM, iSCSI storage.

