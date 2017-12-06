===========================================
 Using SCons with Git Package Organization
===========================================

This page describes use of SCons to build Fermi software under the new
organization in which each package, such as Likelihood, is stored in its
own GitHub repository.  It first covers the typical case of doing a
complete build (target "all") of a superpackage like ScienceTools.  Other
topics include use of supersede area, making builds on public machines at
SLAC, building targets other than "all", and locally-defined SCons options.

Creating Your Work Area
=======================

Prerequisites
-------------
In order to do much of anything you need

- Git.  Version 1.9.5 or later is recommended
- SCons.  At least version 2.1.0
- You probably want repoman as well

Acquiring Fermi source
----------------------
The most common method is to check out the full complement of packages
belonging to the superpackage of interest (ScienceTools, ScienceTools_User
or GlastRelease) via repoman.  See (reference here to workflow page,
repoman section). Alternatively, you can check out the packages yourself
with native git clone commands. This would be tedious in the extreme for a
complete superpackage, but would be appropriate if you only need a few
packages in a supersede area, to be built against a complete installation.
In either case, you will first need a base directory.  cd there before
invoking repoman or doing the git clones.

Externals
---------
In order to build any of the "regular" targets you need access to include 
files, libraries, etc. for the proper versions of the externals required 
by the superpackage you intend to build.  How you acquire them is outside
the scope of this document (though see discussion of --externalsList below
for some help on discovering what you need). The required externals should
be arranged as they are in the Fermi afs area at SLAC.  See
e.g. /afs/slac/g/glast/ground/GLAST_EXT/redhat6-x86_64-64bit-gcc44
Typically the environment variable GLAST_EXT is defined to point to such a
directory (and such an assignment will be assumed in the rest of this
document).

Typical SCons Build Commands and Their Output
=============================================
Note that SCons keeps track of dependencies.  If you ask for a particular 
build product, such as an executable, SCons will additionally rebuild any
of its dependencies which are absent or out of date (e.g., an object file
which is older than the corresponding source).

Building "all"
--------------
Suppose 

- you have used repoman to clone all the source for ScienceTools, under 
  a directory called **ST\_root** 
- SCons is in your path
- GLAST_EXT has been properly defined

Then, if you cd to ST\_root, you can build everything with the command::
 
  scons -C ScienceTools --site-dir=../SConsShared/site_scons --with-GLAST-EXT=$GLAST_EXT all

If your current working directory is something else, the value for the -C
option should be the path to ST\_root/ScienceTools (absolute or relative
from wherever you are).
Nothing else changes. In particular, the value for --site-dir doesn't change 
because it's relative to the directory specified by -C.

The output
~~~~~~~~~~
SCons will create (if they don't already exist) several directories under
ST\_root for build products meant to be public.  Exactly which directories 
are created depends somewhat on the superpackage being built, but they 
always include **bin**, **lib**, **exe** and **include**.  The immediate
results of compile, link, etc., operations are written to a directory
**build** (or a subdirectory) under the package owning the source. Anything
needed for running a program or for building some other package which
depends on the current package (such as public include files or libraries) 
is copied to one of the directories under ST\_root.

Building something else
-----------------------
The command is nearly the same as above; just replace **all** with the target
name.  Note that, as for **all**, you must include the --with-GLAST-EXT
option for all of the following:

libraries
  Builds all libraries that would have gotten built by **all**; that is,
  contents of **lib**

binaries
  Builds contents of **bin** and **exe** (and all dependencies)

include
  Installs all public include files under **include**

Likelihood
  Builds all libraries, test programs, etc., specified in the Likelihood
  SConscript.  Any other package name is similarly treated as a target

Likelihood-libraries
  Builds libraries (and their dependencies) for Likelihood, but not executables.

NoTarget
  Builds nothing at all, but will fail if there are any errors in 
  SConscripts or any other files SCons uses to determine how to build 
  real targets.

Using --supersede
=================
If you're a developer who only needs to modify packages at the top of the
hierarchy (packages which are not dependencies for other packages) or near
the top, build times will be shorter if you make use of the --supersede option.
In the discussion below for concreteness it is assumed that the superpackage 
of interest is ScienceTools, but the same procedure will work for any
superpackage.

Setting up the installation
---------------------------
You'll need all the build products (binaries, libraries, public includes,
etc.)  of the release you wish to build against in the organization SCons
expects to find.  Suppose you decide to put it under a directory
**base\_root**.  You can cd to that directory and check out source using
repoman and build it yourself, using the instructions above to build "all".
Or you may be able to obtain a user tarball and unpack.  Such a tarball
contains *only* public build products; it does not contain source.

Next you need a place to put the packages you'll be modifying (and any
packages using them if some package you are modifying is not at the top
of the hierarchy).  Create a directory, say **supersede\_root**, which is
independent of **base\_root** (neither is under the other).  cd there
and do a git checkout of all the packages you need.    For each package
you intend to change you will probably want to make a branch and develop
on that branch.  It will simplify later operations if all the branches have
the same name.

Building with supersede
-----------------------
Let's assume you have made changes to Likelihood and wish to build it.
cd to **supersede\_root** (maybe not strictly necessary, but at least
avoid a working directory under **base\_root**) and issue a command like this::

  scons -C <absolute-path-to-base_root/ScienceTools> 
     --site-dir=../SConsShared/site_scons --with-GLAST-EXT=$GLAST_EXT 
     --supersede=<absolute-path-to-supersede_root> Likelihood

scons will use Likelihood source in your supersede area and will install
build products there as well.

SCons options
=============
To see all the standard options, just type
::

   scons --help

For Fermi we have added several local options, such as --with-GLAST-EXT 
and --supersede.  To see documentation for all of these, cd to the root
of an installation (suppose it's of ScienceTools) and give the command::

   scons -C ScienceTools --site-dir=../SConsShared/site_scons --help

or, from anywhere
::

  scons -C <absolute-path-to-ScienceTools-package> --site-dir=../SConsShared/site_scons --help

Some useful options
-------------------
The following are standard SCons options:

-k                    Keep going (i.e., do not quit when an error is found)
--clean               Remove specified targets and dependencies               
-c                    Same as --clean

These are locally defined for Fermi:

--with-GLAST-EXT=DIR   Where to find externals. Required whenever building
                       or cleaning "normal" targets, e.g. ones involving 
                       compile.
--ccflags=FLAGS        Pass these (additional) flags to C and C++ compiles
--cxxflags=FLAGS       Pass these (additional) flags to C++ compiles
--supersede=DIR        Root of supersede directory
--rm                   Output at end of output indicating where problems 
                       occurred. Should be used with -k
--externalsList=FILE   Create file in data directory listing all externals
                       required to build "all".  Do not specify 
                       --with-GLAST-EXT when using this option. FILE defaults
                       to externals.extList
--user-release=FILE    Creates tarball of build products. Build must have
                       already been run.  Do not specify --with-GLAST-EXT
                       when using this option.
--source-release=FILE  Creates tarball of source, as if it had been checked
                       out with repoman. Do not specify --with-GLAST-EXT
                       when using this option.
--devel-release=FILE   Creates tarball of build products plus source. 
                       Build must have already been run.  Do not specify 
                       --with-GLAST-EXT when using this option.

For the complete list of standard and local options, issue the --help
command as described above.


Using SCons at SLAC
===================
Certain things are done for you if you're on a SLAC public machine.

1. Externals are already installed.  You can define the environment variable
   GLAST_EXT to be
   ::
      /afs/slac/g/glast/ground/GLAST_EXT/<variant>

   where <variant> depends on the OS of the machine you're logged into. For
   redhat 6 <variant> should be ``redhat6-x86_64-64bit-gcc44``

2. scons version 2.1 is installed, but is unfortunately not the default.
   You can find it at 
   ::
       /afs/slac/g/glast/applications/install/@sys/usr/bin/scons

3. Old release builds (in case you want to use them as a base for work using
   --supersede) can be found on nfs.  Whether the Jenkins builds will also
   be copied to a disk accessible from SLAC public machines is yet to be
   determined.
