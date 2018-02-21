Note on Python2 to Python3 upgrade done for existing actors
======

Conversion
------

Copied from ICS #992, refer long story at http://python-future.org/overview.html

- install the python-future package.
- from the top of your project:

  - ``futurize -1 -n -w .`` 
    (runs stage 1 of futurize, which just fixed print() and imports.)
  - ``git commit`` 
  - ``futurize -2 -n .`` 
    (runs stage2 of futurize, but does not apply the changes. Look at them)
  - ``futurize -2 -n -w .`` 
    (runs stage2 of futurize)
  - ``git commit`` 
  - find and replace all instances of "old_div" in the code. 
    This was put in for all python2 division operators (/ and //). 
    After this, there should be no old_div() calls, and no import of old_div.
  - ``git commit`` 
  - find (by looking at the output of stage2) new "list()" put
    around possible iterators. removed the ones you can.
  - ``git commit`` 
  - examine all other changes from stage2.

Remark:
If you do any io to devices, they probably want bytearrays and not strings. 
Find the read/write calls and add encode('latin-1')/decode('latin-1'). 
latin-1 is used because it leaves all 256 unsigned char values unchanged.


References
------

* `SQR-014 LSST Python 3 <https://sqr-014.lsst.io/>`_

