====================
Developing At SLAC
====================

Prerequisites on RHEL6
----------------------

- Git.  Version 1.9.5 or later is recommended  Available via ``scl enable git19 bash``
- set GLAST_EXT ``/afs/slac/g/glast/ground/GLAST_EXT/redhat6-x86_64-64bit-gcc44``
- SCons.  At least version 2.1.0 ``/afs/slac/g/glast/applications/SCons/2.1.0/bin/scons``
- repoman. 

  - Available as part of this miniconda install: ``/afs/slac/g/glast/ground/GLAST_EXT/redhat6-x86_64-64bit-gcc44/repoman/miniconda2-4.3.30``
  - ``export PATH=$GLAST_EXT/repoman/miniconda2-4.3.30/bin:$PATH``
  
  
Building against a GlastRelease Build
--------------------------------------

We maintain release builds in `/nfs/farm/g/glast/u52/repomanBuild/redhat6-x86_64-64bt-gcc44/GlastRelease`. For development, it is often easier to checkout the package(s) you are directly working on and build against an existing GR build. 

- Create a local work directory
- Checkout your package(s) using git, i.e `git clone https://github.com/fermi-lat/TkrDigi.git`
- cd into your package directory, i.e. `cd TkrDigi`
- Create a branch for your development, `git checkout -b <branchName>`
- Add your new branch to the remote git repo, `git push -u origin <branchName>`
- Modify your code
- Build against an existing GR build

  scons -C <absolute-path-to-base_root/GlastRelease> 
     --site-dir=../SConsShared/site_scons --with-GLAST-EXT=$GLAST_EXT 
     --supersede=<absolute-path-to-supersede_root> TkrDigi
     
- When you are ready to commit your changes back to the remote repository
  
  git add <fileNames>
  
  git commit -m"Commit Message"
  
  git push
  
References
-----------

`repoman <https://fermi-lat.github.io/repoman/>`_

`scons <https://github.com/fermi-lat/doc/blob/master/scons.rst>`_

Building GlastRelease 
---------------------
  
Checking out the L1 branch of GlastRelease from git using repoman
##################################################################
  
- Create a work directory ``mkdir GR-work``
- ``cd GR-work``
- ``repoman checkout --bom GlastRelease L1``

This will checkout all of GlastRelease and its subpackages, using the L1 branch when appropriate.  It will also create a repoman_bom.json file which lists all the git commit tags associated with every package.

Building GlastRelease
#######################

``/afs/slac/g/glast/applications/SCons/2.1.0/bin/scons -C GlastRelease --with-GLAST-EXT=$GLAST_EXT --compile-opt --variant=redhat6-x86_64-64bit-gcc44-Optimized --site-dir=../SConsShared/site_scons all``
