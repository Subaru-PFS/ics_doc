PFS ICS dnsmasq Configuration Conventions
******

- This document is applicable to 'ics_dnsmasq' repository in Subaru-PFS organization
- Follow RFC #2119 for requirement levels

**TOC**

- `General`_

  - `Branch and deployment management`_
  - `Directory and file management`_

- `Contents (definitions in configuration files)`_

  - `Global dnsmasq configurations`_
  - `Site specific dnsmasq configurations`_
  - `Host configurations`_

General
******

In this section, git repository configurations, such as directory, file, and branch, are defined. 

Branch and deployment management
======

Not like normal git branch management way, the 'ics_dnsmasq' repository SHALL 
have only `master` branch for all sites both the production, Subaru summit 
environment as real, and and all AIT or development sites. 
Files and directories of configurations SHALL be categorized into either 
`global` or `site specific`, and `site specific` configuration files or 
directories SHALL be loaded by the dnsmasq daemon using symlink configured 
at each instance, following scheme defined in the 
`Directory and file management`_ section. 

With this scheme, merging contents in normal git branch management way will 
not happen, but it is RECOMMENDED to store conflict free configurations, 
such as pairs of MAC address and host name, in `global` ones not to duplicate 
configurations in multiple `site specific` ones. 

- Many sites have different regulation of IP address assignments, and 
  configuration files in `hosts` directory could be different. 
- Configuring daemon to load `site specific` configuration files or directories 
  using symlink is a task just once per instance, and could be less harmful 
  rather than doing merge configuration files between branches at each time 
  of delivery. 

Directory and file management
======

The 'ics_dnsmasq' repository is organized to be deployed as `/etc/dnsmasq.d/` 
and SHALL NOT contain any file or directory which causes syntax error by 
dnsmasq daemon. Assumed deploy method and operation is to replace entire system 
default installed configuration at `/etc/dnsmasq.d/` into cloned one from 
git repository, `.git/` directory (`/etc/dnsmasq.d/.git/`) will be ignored 
by the dnsmasq daemon on parsing contents in `/etc/dnsmasq.d/` and this way 
will work fine. We MAY have some notes or comments in files placed in the 
repository, escaping or commenting out is REQUIRED for any file in the 
repository even if the entire file is just a text file without any line for 
configuration (like readme). Placing such files as dot filename, like 
`.README.rst`, is NOT RECOMMENDED. 

To have `site specific` configuration files and directories, system 
administrator of each instance SHALL 

- Configure dnsmasq to load `/etc/dnsmasq.d/*.conf` by editing 
  `/etc/default/dnsmasq` file to have a line - 
  `CONFIG_DIR=/etc/dnsmasq.d,*.conf`. 
- Make symlink for `site specific` dnsmasq configuration, from 
  `/etc/dnsmasq.d/dnsmasq-site.\<site\>` to `/etc/dnsmasq.d/dnsmasq-site.conf`.

In the 'ics_dnsmasq' repository, we SHALL place configuration files as:

`host-mac` directory
  Place files in a name of 'target'.conf, which contain lines of a pair of 
  MAC address and its hostname as `xx:xx:xx:xx:xx:xx,hostname` format.
  Only `global` configuration is allowed for this part, all paris used at 
  every sites SHALL be placed here. 
`hosts.\<site\>` directory
  Place files in a name of 'target'.conf, which contain lines in a format 
  used for `hosts` file as `IP_address hostname(s)`.
  This directory SHALL be configured to be loaded by `dnsmasq-site.\<site\>` 
  file, but NOT by symlink. 
`dnsmasq.conf` file
  This file SHALL have all configurations specified in the 
  `Global dnsmasq configurations`_ section, and SHALL NOT have any configuration 
  not described in the section. 
  In other words, any global configurations SHALL be described in the 
  `Global dnsmasq configurations`_ section with its necessity. 
`dnsmasq-site.\<site\>` file
  This file SHALL have all configurations specified in the 
  `Site specific dnsmasq configurations`_ section, and SHALL NOT have any 
  configuration not described in the section. 

Any other directory or file with configurations SHALL NOT be added or 
placed into the 'ics_dnsmasq' repository. 
As in `Global dnsmasq configurations`_ or 
`Site specific dnsmasq configurations`_ section, 
it is possible to add new separated 
file for groups of configurations, such as PXE as DHCP option for flagged 
hosts, but such file SHALL be added in the list above before added into 
the `ics_dnsmasq` repository. 

'target' name used for configuration files SHALL be based on acronyms listed 
in the product tree of the PFS, such as PFI (Prime Focus Instrument), MCS 
(Metrology Camera System), or RCU (Red Camera Unit), with number(s) attached 
for identifying multiple instances. Commonly used shorter names like r1 for 
RCU1 are NOT RECOMMENDED, not to confuse team members. 
Considering replacements by maintenance, especially for hardware replacement 
consisted with several hardware and control boxes, it is RECOMMENDED to 
break configurations into files by domains to be used, such as a set of 
control computers and hardware for cameras (like FCC) in PFI, or a piepan of 
each cryostat in SpS. 

'site' name used for `site specific` configuration files SHALL be a commonly 
used short name of a site in all lower cases, such as 'subaru', 'ipmu', or 
'jhu'. 

Files in two directories, `host-mac` and `hosts`, SHALL be the same file name 
for the same target. Like, for host `mac` with `ab:cc:ef:01:23:45` and 
`10.123.45.67` in `mac` target category, configurations will be done as 
`ab:cc:ef:01:23:45,mac` in `host-mac/mac.conf` and `10.123.45.67 mac` in 
`hosts/mac.conf`. 

Contents (definitions in configuration files)
******

Some of this section is RECOMMENDED for instrument development sites (or 
branch in git repository) but is NOT REQUIRED. 

Global dnsmasq configurations
======

Following configurations SHALL be included. 
`xxx` in configurations SHALL be replaced with real values. 

- `dnsmasq` configuration files' definitions

  - `dhcp-hostsfile=/etc/dnsmasq.d/host-mac`

- DNS

  - `local-ttl=900`: `local-ttl` is a configuration of TTL (in seconds) in 
    reply from dnsmasq service and used for cache at requester. Default is `0` 
    which means requester (DNS client) SHOULD NOT cache replies. This is to 
    reduce load of dnsmasq service and network traffic. 
  - `expand-hosts`: This is required to build FQDN from `addn-hosts` 
    configuration.
  - `domain-needed`: This is required not to break upstream DNS server.

- DHCP

  - `log-dhcp`: This makes dnsmasq to log all DHCP requests and replies, which 
    is useful for issue handling and trouble shooting. 

Following configurations MAY be included.

- DNS

  - `log-queries`: This makes dnsmasq to log all DNS queries into a log file, 
    but most of logs are useless. 
  - `bogus-priv`: In production, IP address range is not in private IP ranges, 
    this configuration will not affect to anything nor is not harmful. 
    But could be useful in some development sites. 

- DHCP

  - `dhcp-sequential-ip`: This is to lease DHCP IP address in sequential but 
    not determining by a hash of the client's MAC address. 
  - `dhcp-lease-max`: is default to 1000 and could be enough, but we MAY limit 
    below than the default. 
  - `dhcp-authoritative`: In the PFS network, the dnsmasq service is the only 
    one DHCP server on a network, and this should be set (but could work 
    without this configuration). 

Following configurations SHOULD be included when PXE/TFTP is required for 
operation, such as SpS/BEE. These configurations MAY be added as a separated 
configuration file at the top level directory in the `ics_dnsmasq` repository. 

- `dhcp-option-force=xxx`
- `dhcp-boot=tag:pxe,pxelinux.0`
- `enable-tftp`
- `tftp-root=/xxx`
- `tftp-secure`

Site specific dnsmasq configurations
======

Following configurations SHALL be included. 
`xxx` in configurations SHALL be replaced with real values. 

- `dnsmasq` configuration files' definitions

  - `addn-hosts=/etc/dnsmasq.d/hosts.\<site\>` to include hostname to IP 
    address configuration.

- DNS

  - `txt-record=xxx,xxx`: This txt record is REQUIRED for operation of FITS 
    name building (as for now) to identify at which site the system is working. 

- DHCP

  - `domain=xxx`: for default domain used in the site
  - `dhcp-range=xxx`: for DHCP configurations. At least two lines are REQUIRED, 
    one for all range of assignable IP addresses (for IP addresses, which are 
    not included in any of lines, are not assigned even if specified in 
    dnsmasq configurations), and one with `tag:!known` option to specify 
    temporary IP addresses. 
  - `dhcp-option=option:ntp-server,xxx`: for configuration of NTP server. The 
    NTP server MAY be by Subaru but PFS could have its own. 

Following configurations MAY be included.

- DNS

  - `no-resolv`, `server=xxx`: In production, by default, upstream DNS server 
    configuration is to be specified in `/etc/resolv.conf`, but these two 
    configurations could be added just in case. 

Host configurations
======

Host configurations are defined by two files in both `hosts.\<site\>` and 
`host-mac` 
directories, which define IP address and MAC address against hostname 
respectively. Hosts are categorized into two, one SHALL NOT depend on DHCP 
and SHALL be configured as static at OS such as network switches or VM hosts 
which need to run before the dnsmasq service on a VM client starts, 
and another is all others most of which MAY work both with DHCP or static. 
For both cases, hosts SHALL be configured in the dnsmasq service as follows. 

- Every pairs of IP address or MAC address to hostname SHALL be included in 
  configuration files. Even for ones configured as static, a pair SHALL be 
  included. This is for DNS resolv, recording of hosts, and in case of 
  trouble (to assign IP address by DHCP for these hosts). 
- All NICs on computing hardware SHALL be included in configuration files 
  in `host-mac` directory. A hostname for additional NIC SHALL follow the 
  main one, such like `vmhost1b` for a host named as `vmhost1`. 
- A hostname SHALL be fixed to function of target component but not hardware, 
  and SHALL be taken from its function. This means a hostname assigned to a 
  function, like BEE of RCU1, SHALL NOT be replaced on replacement of hardware 
  by maintenance. 

  - VM hosts MAY be named by their hardware, such as `r410-1`, but service 
    oriented names (or name fixed to function) SHALL be used for entries in 
    DNS/DHCP configuration files.

Also these hostnames are RECOMMENDED to consider following points.

- 'hostname' MAY contain '-' for separations between subparts, but SHALL NOT 
  use '_' for separations (RFC violation).
- Subparts of 'hostname' is RECOMMENDED to be well defined name in the PFS 
  product tree, such as `bcu1` but not just `b1`, to make hostname to be self 
  described. 

For configuration files in `hosts.\<site\>` directory, 
which contains pairs of hostname 
and IP address in hosts format, every lines are RECOMMENDED to consider 
following points.

- Only one hostname, from which defined in `host-mac` as pairs of hostname and 
  MAC address, is defined for one IP address. 'dnsmasq' takes first 
  definition (first line or first item in a line), but ignores any of 
  followings as double defined for fixed IP address assignments of DHCP. 
- Multiple hostname MAY be defined for DNS to be used for having alternative 
  name of a target to be connected from control software. 
- These configuration files SHALL NOT be changed on replacing hardware for 
  maintenance, and SHALL be static over the entire period of operation except 
  for an event of reorganization over the entire network and subnet. 

Within PFS LAN, several physical servers may have multiple NICs and could be 
connected to a network switch in bonding. For hardware control computers, 
there is almost no need to have such high bandwidth connection, and requirement 
or necessity of these configuration may be limited to physical servers at 
CB2F, such as VM hosts. For these physical servers, it is RECOMMENDED to 
configure as follows.

- Every hosts are RECOMMENDED to be configured as static but not DHCP, 
  especially for bondX network interface. 
- All MAC addresses of physical NICs SHALL be recorded into a corresponding 
  `host-mac` configuration file. 

