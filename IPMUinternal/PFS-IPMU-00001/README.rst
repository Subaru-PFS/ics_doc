PFS-IPMU-00001: Internal design for PFS IPMU online services
======

This document contains internal design notes for PFS IPMU online services, 
and divided into following notes.

* `hardware components design and deployment <hardware.rst>`_

  * `2-314 rack assignment <2-314-rack.rst>`_
  * Computing resource lists

* Server components

  * `Infrastructure overview (storage, VM host) <infra.rst>`_
  * `Network assignment <network.rst>`_
  * `Services overview <services.rst>`_

* Manuals and notes on operations and maintenance

  * `Service cold start and full shutdown <startdown.rst>`_
  * `Service setup and configuration <setup.rst>`_
  * `Configurations and maintenance <maintenance.rst>`_
  * `System monitoring and notification <monitoring.rst>`_

* Misc documents (from old internal wiki)

  * `Making CD/DVD <cdburn.rst>`_


Legends in this document
------

* ``inline literal`` means canonical service name, like ``ntp``, or ``pfsdisk``
* *emphasis* means hostname without tailing revision number of VM guest 
  reinitialization (or switching to new one), like *stretch*, or *pd*

  * VM hostname has tailing `-vm`, so digit before it is hardware component ID
  * Tailing digits could be more than two, like version number of hosted 
    service like JIRA
  * In configurations, need to be referenced by canonical service name

* `solid text` means command or option

