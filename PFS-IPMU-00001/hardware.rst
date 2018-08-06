Hardware components design and deployment
======

This page is to show overview of hardware component design. 
Refer list of documents in 'Server components' section of 
`README <README.rst>`_ for (detailed) implmentation and configuration. 

ToC

* `Key concepts`_
* `Storage service`_
* `VM computing resource`_

Key concepts
------

To make system as redundant as possible in reasonable initial (e.g. hardware) 
and operational cost, layers of system are categorized into three and designed 
following requirements placed to each layer, except for devices under testing 
(under investigation for installation) or for special use case (PCIe SSD for 
DB test server or GPGPU server). 
Also to make system recovery procedure easy and not to propagate single 
hardware issue to wider system/layers, VM based system design is used as 
much as possible to separate hardware configuration and operational software. 

1. key infrastructure

  * System wide shared storage services and their end points (like NFS, WebDAV) 
    are categorized into this layer.
  * At minimum N+2 configuration is required for storage space, 
    like RAID1 with hot spare (N+2), RAID6 with hot spare (N+3). 
    Controller box is better to be in redundant configuration (but currently 
    no).
  * Hosting computer(s) are better to be configured as redundant, especially 
    for power supply (redundant with connecting to different UPS), system 
    storage (RAID1), and are required to be configured using automated system 
    like ansible.
  * Storage service end points shall be VM guests and configured using 
    automated system like ansible, to make relocation (at hardware failure) or 
    reinstallation/reconfiguration (at the most serious case) work easy and 
    quick without consulting to detailed manual etc. 

2. computing nodes

  * Conputing resources to run VM guests are categorized into this layer.
  * Any hardware shall be replacable without special software configuration 
    for VM guests running on them (like just by virt hot migration). 
  * Hardware configuration is better to be redundant in the simplest way, 
    like RAID1 for its system disk, dual power supply, ECC memory, but not 
    a requirement. (mostly, RAID1 and ECC are used, but one of dual power 
    supply is connected.)
  * Operating system configuration shall be performed using automated system 
    like ansible. 

3. remaining important single point of failure (SPOF)

  * This category is a collection of SPOF which are not easy to be implemented 
    as redundant. 
  * SPOFs in the system are listed with possible (implemented) mitigation(s). 
  * Network switch: spare switches are in storage
  * iSCSI storage controller (RAID6): no mitigation for now. hardware is 
    configured as single backplane and single controller.
  * Main storage box (RAID1): backup box with 10x 3.5in HDD is in storage, HP 
    RAID controller keeps RAID configuration in HDD so no reconfiguration is 
    required on replacing control computer. 

Storage service
------

PFS infrastructure provides six storage resources as follows:

* /virt: VM guest disk image pool. Exposed as NFS and all disk images are in 
  raw format (no qcow2 etc. is allowed). 
* /server: Service data storage. Exposed as NFS.
* /simdata: Instrument development use, like ICS or observation simulation. 
  Exposed as NFS. 
* /data1: IPMU internal general storage service. Exposed as NFS and smb. 
* /backup: Periodic differential backup storage, offline from normal servers 
  and only accessible from VM guest for backup operation due to security reason 
  (not to be affected by malware etc. through shared channel). 
* /arch1: development archive storage. Exposed as web or webdav (and NFS 
  internally).

First four areas are critical for system operation, so 

* hardware for these four area are configured as redundant as possible, and 
  also keep enough level (more than N+2) of failure protection on spin up 
  device (HDD). 
* keep periodical backup to (semi-)offline system on different hardware, to 
  make recovery possible even at serious data damage. 

VM computing resource
------

Except for performance (like bogomips-ish things), resources are matter of 
number of cores (and memories) provided through the pool, except for some 
special case on CPU instruction set (like AES-NI) or network configurations 
(bridges are provided to which network). 
On this point, we just have computing resources as easily replacable pool, 
but not a critical component. 

