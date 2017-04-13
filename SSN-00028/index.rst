PFS ICS dnsmasq Configuration Conventions
******

- This document is applicable to 'ics_dnsmasq' repositogy in Subaru-PFS organization
- Follow RFC #2119 for requirement levels

General
******

In this section, git repository configurations, such as directory, file, and branch, are defined. 

Directory and file management
======

The 'ics_dnsmasq' repository is organized to be deployed as `/etc/dnsmasq.d/` 
and SHALL not contain any file or directory which causes syntax error by 
dnsmasq daemon. Assumed deploy method and operation is to replace entire system 
default installed configuration at `/etc/dnsmasq.d/` into cloned one from 
git repository, `.git/` directory (`/etc/dnsmasq.d/.git/`) will be ignored 
by the dnsmasq daemon on parsing contents in `/etc/dnsmasq.d/` and this way 
will work fine. We MAY have some notes or comments in files placed in the 
repository, escaping or commenting out is REQUIRED for any file in the 
repository even if the entire file is just a text file without any line for 
configuration (like readme). Placing such files as dot filename, like 
`.README.rst`, is NOT RECOMMENDED. 

In the 'ics_dnsmasq' repository, we SHALL place configuration files as:

`host-mac` directory
  Place files in a name of 'target'.conf, which contain lines of a pair of 
  MAC address and its hostname as `xx:xx:xx:xx:xx:xx,hostname` format.
`hosts` directory
  Plase files in a name of 'target'.conf, which contain lines in a format 
  used for `hosts` file as `IP_address hostname(s)`.
`dnsmasq.conf` file
  This file SHALL have all configurations specified in the 
  `Global configurations`_ section, and SHALL NOT have any configuration 
  not described in the section. 
  In other words, any global configurations SHALL be described in the 
  `Global configurations`_ section with its necessity. 

Any other directory or file with configurations SHALL NOT be added or 
placed into the master branch of 'ics_dnsmasq' repository. 

'target' name used for configuration files SHALL be based on acronyms listed 
in the product tree of the PFS, such as PFI (Prime Focus Instrument), MCS 
(Metrology Camera System), or RCU (Red Camera Unit), with number(s) attached 
for identifying multiple instances. Commonly used shorter names like r1 for 
RCU1 are NOT RECOMMENDED, not to confuse team members. 

Branch management
======

Not like normal git branch management way, the 'ics_dnsmasq' repository makes 
the `master` branch to be defined as the one used at the production - Subaru 
summit environment as real. Other branches are to be used at each development 
site with a name of each site, such as `LAM` and so on. 

Merging from one branch to another SHOULD happen at points of hardware 
deliveries, but can be perfomed well in advanced. It is RECOMMENDED to have 
separated files in `host-mac` and `hosts` directory per each hardware 
delivery. For most cases, actual procedure of merging will not be by normal 
ways using git, but just copied and newly added to the new branch is 
RECOMMENDED with including a commit hash and a branch name of the origin. 

- Many sites have different regulation of IP address assignments, and 
  configuration files in `hosts` directory could be different. 
- Getting the exact files for one delivery with simple diff between two 
  points of commit history is difficult for some instrument development sites, 
  it is simple just to copy with modification into a branch for target of 
  delivery rather than having complexed flow of git commands. 

Contents (definitions in configuration files)
******

Global configurations
======


IP address range assignments
======


Host configurations
======


