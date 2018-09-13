Configurations and maintenance
======

ToC

* `LDAP account`_
* `LDAP server`_
* `JIRA`_
* `Special cases on replacing host`_
* `Let's encrypt / certbot`_
* `Updating web services`_
* `Mailman`_

LDAP account
------

Use 
`LDIF generation script <https://github.com/Subaru-PFS/pfs_infra_ipmu.git>`_ 
for operation (`ldap-admin` in repository). 
UID number registration (listing) file is required for new account generation, 
use one at `/server/admin/github-scripts/pfs_infra_ipmu/ldap-admin/`. 

* Create target list file (see command help below)
* Place photo jpg file as username.jpg
  (automatically loaded into ldif file for adding new user)
* Execute `create-ldif.pl` script
* Execute `.cmd` file generated to run ldapadd/ldapmodify
* Load on services

  * JIRA will reload per about 30min
  * Bugzilla need to run `contrib/syncLDAP.pl -d`

::

 $ ./create-ldif.pl
 <script> <options> target_list
 options:
    --reset : execute reset (password)
    --group=<str> : enable LDAP group specified by <str>, multiple accepted
    --jira : enable JIRA access
    --shell : define user as shell access group (cannot be used with --reset)
 target_list is a filename with lines by "<email> <username> <fullname>"
    uid is taken from '00uid.list' file (last +1)
    default gid is from opt_gid (2000)
    fullname is an option, from third to the end when defined

LDAP server
------

* config root is 'cn=admin,cn=config'
* on upgrade/move, copy /etc/ldap/slapd.d and /var/lib/ldap

JIRA
----

Currently executed upgrade procedure is:

* make new VM guest with jirapipe-<version> as hostname
* make new directory /server/service/jirapipe-<version>, mount it as /service
* download new package of JIRA software
* install JIRA software

  * install target to /service/jira
  * JIRA software data to /service/jira-data

* copy jira-data from old /server/service/jirapipe-<version>/jira-data
* edit conf/server.xml to point /jira/ (path attribute of Context)

Special cases on replacing host
------

Followings are special case on replacing host using different hostname or IP 
address, which will require additional modification(s) on existing services. 

* NTP server: After replacing ntp server, change configuration not only in 
  `hosts-ipmu` but also ntp-server dhcp-option line in `dnsmasq-site.ipmu`. 
* NFS server for ``/virt``: VM host physical machines are required to be 
  configured as mounting ``/virt`` with IP address but not service canonical 
  hostname. Therefore, change ansible configuration and switch NFS mount 
  configuration in fstab with new IP address. 

Let's encrypt / certbot
------

Server certificates for PFS servers at IPMU are acquired using certbot from 
`Let's Encrypt <https://letsencrypt.org/>`_, and used by httpd and mail 
systems. 
Certifications are automatically updated via periodic execution of certbot 
(configured into cron by its package). Although httpd(s) could be configured to 
be auto-reloaded through update process, mail systems are not, so we need to 
force reload to mail systems (and so on, if any addition we will introduce). 
For these, we configure cron job(s) to reload certification(s) instead of by 
certbot. Updates are performed per ~2 monthes, and certificates have 3 monthes 
period, cron job(s) are configured twice per month. 
These cron job(s) are implemented in ansible with service linked flags. 

Updating web services
------

Wordpress (for blog) and MediaWiki (for internal wiki) are used and require 
manual update at some point (not solid requirement, but better for recovery 
or backup). 
There is some tools and procedures within PFS IPMU servers to make works easy. 

* Wordpress

  * Download package of updated version from official site to a directory for 
    each service
  * Extract as `wp-<version>`
  * Execute `links.sh` in a directory of extracted new version
  * Re-link `blog` to a directory of new version

* MediaWiki

  * Do similar to wordpress
  * No `links.sh` is provided yet, do manual operation...

Mailman
------

Operations of mail lists, especially to change something, are better to be 
performed through web. If you modified something locally via shell, you need 
to check permissions of list configuration files to make web and/or mail 
server priviledge can read/modify these configuration files. 

To change the last post ID (added to subject):

::

  $ sudo -u list bin/withlist allhands
  >>> m.Lock()
  >>> m.post_id = <newid>
  >>> m.Save()

On permission error:

::

  $ sudo -u list bin/check_perms -f



