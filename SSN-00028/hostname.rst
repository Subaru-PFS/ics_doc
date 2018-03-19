PFS ICS hostname Conventions
******

- This document is applicable to 'hostname's used in PFS ICS
- Follow RFC #2119 for requirement levels

**TOC**

- `Naming hostname`_
- `Keywords`_
- `Remarks and exceptions`_

Naming hostname
======

- PFS ICS hostnames SHALL be named as `\<feature\>-\<subsystem\>` format, 
  like `bee-b1` for BEE used in BCU1. 
- Both `\<feature\>` and `\<subsystem\>` SHALL be named in some charactors of 
  lower case alphabet with following numerical charactors for multiple 
  instances. 

  - Numeric character(s) SHALL follow alphabetical characters only to 
    indicate multiple instances such as four spectrograph modules 
    (sm1, sm2, sm3, and sm4). Namely, any numeric character SHALL NOT follow 
    alphabetical character(s) in case of single instance such as pfi.
  - Following numerical charactors MAY be 2 digits in `\<feature\>` for 
    largely multiplated devices, but SHALL be as short as possible. 

- Alphabet characters in `\<subsystem\>` SHALL be selected from ones listed 
  in `Keywords`_ section which are the acronyms of PFS subsystems.
- Alphabet part of `\<feature\>` is RECOMMENDED to be selected from acronyms 
  listed in the PFS product tree.

Keywords
======

This section lists keywords to be used in the `\<subsystem\>` part, 
and the following `\[1-4\]` part defines the possible range of digit to 
indicate multiple instances. 
If this list does not include a keyword that needs to be used, 
one SHALL add it to this list before using it.

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

Remarks and exceptions
======

This section lists various remarks and typical exceptions from listed above. 

- NFS service is assumed to be capable reorganizing into separated hosts. 
  Adding individual target hostname for each target (directory) is assumed, as 
  listed below:

    - nfs-ics: master alias as physical host which proxy the largest total 
      bandwidth disk IO stream
    - nfsv-ics: for /virt
    - nfss-ics: for general purpose storage
    - nfsd-ics: for detector data storage

