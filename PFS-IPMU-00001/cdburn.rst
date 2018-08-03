Making CD/DVD at IPMU service
------

Following hosts has writable drive for CD/DVD

* dlb8-vm

Procedure
======

* First, copy target iso image to host local which you will burn disk.
* Execute wodim as 'wodim -v dev=/dev/scd0 imagefilename.iso'

