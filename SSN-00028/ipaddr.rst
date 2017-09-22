PFS IP address (CIDR) assignment at production
******

- This document is applicable to IP address assignment to subsystems at production.
- CIDRs are assigned to subsystems in this document, individual assignments are registered in ics_dnsmasq repository.

Basic concepts
======

- Assign IP address by CIDR block to subsystem, to enable simple filtering at gateway router (e.g. allow/deny by block)
- Take reasonable number of spare addresses to each subsystem
- Assign a block to SM1-4, and allow SMs to have human readable numbers, like bee-bX to 'i + 30 x'
- Not to cut address range smaller than /28

CIDR assignments
======

- **.164.0/28 (.164.1 - .164.15)**: Network switches (.164.1 for Subaru core switch)

  - Subaru core, CB2F switch, SpS (distribution, access x4 for SM1-4), Cs, PF

- **.164.16/28 (.164.16 - .164.31)**: Services to external

  - ssh gateway
  - FTP server for data transfer

- **.164.32/27 (.164.32 - .164.63)**: CB2F infrastructure

  - VM host server
  - storage (iSCSI data x8, management x2)
  - NFS server
  - support devices at CB2F (PDU, KVM)
  - management (iDRAC) for CB2F hosts

- **.164.64/27 (.164.64 - .164.95)**: CB2F ICS services

  - ICS MHS and support services
  - CB2F non-hardware actors (SSC, STH, MAC etc.)

- **.164.96/27 (.164.96 - .164.127)**: PFI

  - Device in PFI
  - Cal-lamp
  - Cobra FPGA (1st stage)
  - CB2F hosts, incl FPS, MPS, AGCC

- **.164.128/26 (.164.128 - .164.195)**: (kept as spare)
- **.164.192/28 (.164.196 - .164.207)**: (kept as spare)
- **.164.208/28 (.164.208 - .164.223)**: MCS (Cs)
- **.164.224/28 (.164.224 - .164.239)**: landfill
- **.164.240/28 (.164.240 - .164.255)**: DHCP work range
- **.165.0/25 (.165.0 - .165.127)**: SM1-4

  - **.165.00 - .165.29**: SM1
  - **.165.30 - .165.59**: SM2
  - **.165.60 - .165.89**: SM3
  - **.165.90 - .165.119**: SM4
  - **.165.120 - .165.127**: (kept blank)

- **.165.128/27 (.165.128 - .165.159)**: SCR control

  - IR3 devices (UPS, etc.)
  - SCR glycol/chiller control
  - SCR temperature control
  - SCR safety monitoring and surveyllance camera
  - SCR control (e.g. room light)

- **.165.160/27 (.165.160 - .165.191)**: SCR support

  - 5th rack VM host and management
  - SpS/SCR management actors
  - 5th rack emergency suppor tools
  - SM roughing pump control

- **.165.192/28 (.165.192 - .165.207)**: SCR monitoring

  - SCR AUX telemetry device (e.g. raspi)

- **.165.208/28 (.165.208 - .165.223)**: (kept as spare)
- **.165.224/27 (.165.224 - .165.254)**: (kept as spare)

