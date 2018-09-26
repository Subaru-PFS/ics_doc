Service setup and configuration
======

Following repositories are used for service setup and configuration. 

* `preseeded iso image <https://github.com/Subaru-PFS/ics_ansible/tree/master/misc/debian-preseed>`_ 

  * To dealt with storage device name, use different image for 
    physical (``/dev/sda``) and virtual (``/dev/vda``; used to indicate virtio)
  * non-official firmware included source iso image is recommended. 
    including custom firmwares into generated iso is future issue.

* `ansible playbook <https://github.com/Subaru-PFS/ics_ansible>`_
* `service configuration <https://github.com/Subaru-PFS/pfs_ipmu_config>`_

  * configurations are used with simply replacing configuration directory to 
    a target branch of repository, like cloning branch into 
    ``apache2/site-enabled`` after (or during) configuration by ansible.

Current status on ansible
------

Following hosts are fully configured by ansible playbook (real ones are 
configured by ansible without any manual operation): 

* storage hosts (*pd*, *pc*, *sd*)
* landfill services (only for template)
* ``rsyslog``

Following hosts are fully configurable by ansible playbook, but need some 
manual operation: 

* (TBC)

Following hosts are currently under development and/or confirmation:

* external servers (``pfs``, ``pfspipe``, ``pfsarch``)
* database servers (``mysql``, ``pgsql``)

Following hosts are not planned yet: 

* JIRA


