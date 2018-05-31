====================
Developing At SLAC
====================

Prerequisites on RHEL6
----------------------

- Git.  Version 1.9.5 or later is recommended  Available via ``scl enable git19 bash``
- SCons.  At least version 2.1.0 ``/afs/slac/g/glast/applications/SCons/2.1.0/bin/scons``
- repoman. 

  - Available as part of this miniconda install: ``/afs/slac/g/glast/ground/GLAST_EXT/redhat6-x86_64-64bit-gcc44/repoman/miniconda2-4.3.30``
  - ``export PATH=$GLAST_EXT/repoman/miniconda2-4.3.30/bin:$PATH``
  
Checking out the L1 branch of GlastRelease from git using repoman
-----------------------------------------------------------------
  
- Create a work directory ``mkdir GR-work``
- ``cd GR-work``
- ``repoman checkout --bom GlastRelease L1``

This will checkout all of GlastRelease and its subpackages, using the L1 branch when appropriate.  It will also create a repoman_bom.json file which lists all the git commit tags associated with every package.
