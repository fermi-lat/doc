===========================================
 Using SCons with Git Package Organization
===========================================

This page describes use of SCons to build Fermi software under the new organization in which each package, such as Likelihood, is stored in its own GitHub repository.  It first covers the typical case of doing a complete build (target "all") of a superpackage like ScienceTools.  Other topics include use of supersede area, making builds on public machines at SLAC, building targets other than "all", and locally-defined SCons options.

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
  Builds contents of **bin** and **exe**

include
  Installs all public include files under **include**

Likelihood
  Builds all libraries, test programs, etc., specified in the Likelihood
  SConscript.  Any other package name is similarly treated as a target

Likelihood-libraries
  Builds libraries (and dependencies) for Likelihood, but not executables.

NoTarget
  Builds nothing at all, but will fail if there are any errors in 
  SConscripts or any other files SCons uses to determine how to build 
  real targets.

