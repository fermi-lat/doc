
A conda/conda-forge primer
==========================

This is a short introduction to conda/conda-forge. It is not complete,
but should provide some information to get started.

Intro and definitions
---------------------

Conda
~~~~~

Conda is a package and dependency manager. It was created by Anaconda,
Inc., but it can live on its own (it is open source). It allows the
distribution of binaries for Linux, Mac and Windows, and it manages
dependencies, versions and so on. It supports environments (similar to
``virtualenv`` of python, but contrary to the latter conda environments
are completely indendent from each other).

From the user point of view, it behaves like other package managers like
``brew`` or ``apt-get`` or ``yum``. However, contrary to these
solutions, it is platform independent.

From the developer/distributer point of view, it simplifies the release
because you need to make one release for Linux (not one per
distribution), one for Mac (which will work on any decently modern Mac)
and one for Windows.

Conda-build
~~~~~~~~~~~

``conda-build`` is the build system of conda, and it is distribued as a
conda package. A typical user do not need to have the ``conda-build``
package. Only the build environment needs to have ``conda-build``.

Anaconda
~~~~~~~~

Anaconda is a product made by Anaconda, Inc. and it is essentially a
bundle of packages for ``conda``. It contains a lot of packages for all
sorts of uses, mainly geared towards data analysis though.

Miniconda
~~~~~~~~~

Miniconda is a minimal set of ``conda`` packages that allows the user to
use the ``conda`` package manager. It is a (very) small subset of the
``Anaconda`` distribution.

Channels
~~~~~~~~

A channel is a repository of binary packages that conda can access.
There are default channels managed by Anaconda, Inc. and there are a
pletora of custom channels. Creating a channel is free thanks to
Anaconda Cloud, a hosting service provided by Anaconda Inc. Opening an
account on Anaconda Cloud is free and it has a 3 Gb limit (free) or 10
Gb limit (paid). It is also possible to deploy your own conda channel on
your own server (super easy, look here
https://conda.io/docs/user-guide/tasks/create-custom-channels.html ).

Conda-forge
~~~~~~~~~~~

Conda-forge is an independent organization with no affiliation to
Anaconda Inc. It provides:

-  From the user perspective, a conda channel (``conda-forge``)
   containing many packages built in a consistent way and only depending
   on other packages within the same channel
-  From the developer perspective, it provides a service where conda
   packages can be built using services like Travis CI and Circle CI,
   and it gives tools (like the ``toolchain`` package) which allow for a
   self-consistent way of building packages. Packages built with the
   ``conda-forge`` tools can be hosted on the ``conda-forge`` channel,
   or in any user channel.

Conda from the user perspective
-------------------------------

The typical interaction of a user with conda is just to run a bunch of
``conda install [something]`` commands, optionally adding a specific
channel (for example ``conda install -c channel [something]``). It will
also typically use environments:

Conda environments
~~~~~~~~~~~~~~~~~~

A conda environment is a fresh conda installation where the user can
install stuff. Differerent environments do not influence each other, in
fact, in a given shell session only one environment can be active.

Caveat: never install any software in the root environment. Always
create first a new environment for it:

::

    > conda create --name myenv
    > source activate myenv
    > conda install [whatever]

For example, at the time of writing installing ``ipython`` in the root
will cause it to propagate to all environments no matter which version
you have in the environment.

Conda from the developer perspective
------------------------------------

Conda build allows to:

-  build once for any Linux, Mac, or Windows. The package can support 1,
   2 or 3 platforms (no need to always support all three)
-  declare the dependencies of the package, with or without pinning
   versions
-  define build matrices (python2/3, numpy versions, openMP or not...)
-  all the produced binaries are fully relocatable, i.e., they can be
   installed by any user on any conda environment using
   ``conda install``
-  the build can be tested as part of the build process so that problems
   can be caught early

Releasing a package on conda
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need to define a recipe, which is usually composed of these 2 files:

-  ``meta.yaml`` is a YAML file defining the source of the code (usually
   a github or a ``pypi`` URL), dependencies and build matrix. It also
   defines 2 different environments with their own independent
   requirements: the build environment where the build is executed, and
   the run environment where the runtime dependencies are listed. Only
   dependencies listed in the latter are going to be installed in the
   user environment as part of the ``conda install`` process. You can
   find instructions on how to make the ``meta.yaml`` for conda-forge
   here: https://conda-forge.org/docs/meta.html.

-  ``build.sh`` is a script which contains the procedure to build the
   package. It can be as simple as
   ``python -m pip install --no-deps --ignore-installed .`` or as
   complicated as you need it to be. Within ``build.sh`` you can use all
   the tools you need (``cmake``, ``make``, ``scons``, whatever) as long
   as they are available in the build environment (i.e., you need to
   list them as build dependencies, except ``make`` which is provided by
   default). For example, typically for a C++ package you would use the
   usual ``./configure``, ``make`` and ``make install`` thing. Using
   Bash if/else statements you can also distiguish between Linux and Mac
   commands, if necessary (see for example here:
   https://github.com/conda-forge/root5-feedstock/blob/master/recipe/build.sh).
   Useful things to know:

   -  If a compiled package, you need to install the package under
      ``${PREFIX}``, so typically you should run
      ``./configure --prefix=${PREFIX}`` or
      ``cmake -DCMAKE_INSTALL_PREFIX=${PREFIX}``

If you want to build for windows as well, you can provide also a
``build.bat`` file which is executed in the build environment under
windows.

You can also insert a test section in the yaml file, or provide a
``test.sh`` script for tests. These tests will be run in the test
environment (which can have additional dependencies than the run
environment).

You need to put these files in a place on your computer (most likely, a
git repository). In particular, if your pakcage is named ``mypackage``
and the work directory is ``work_dir``, you need to put the files under
``work_dir/mypackage``. Then, in ``work_dir``, you run
``conda build mypackage`` and the build starts.

The ``conda build`` process
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The command ``conda build`` wraps your own build procedure, the one you
wrote in ``build.sh`` (or ``build.bat`` for Windows). In particular:

1. It starts by creating a build environment containing all the
   dependencies listed in the ``build`` section in the ``meta.yaml``
   file in a directory with a long path name. Executables and libraries
   will be built with an RPATH (see https://en.wikipedia.org/wiki/Rpath)
   in their header pointing at that long path name. Such path name will
   be overwritten by ``conda install`` upon installation with the real
   path of the conda installation of the user, wherever it might be.
   This is essential to make your products relocatable. Then,
   ``conda build`` will activate the build environment and execute your
   build script. Upon success, it will scan the build environment
   (pointed at by the ``${PREFIX}`` env variable in the build
   environment) for any new product that was not present before running
   the script. Whatever your script have put into ``${PREFIX}`` will be
   part of your package. These products will be first searched for any
   hard coded reference to ``${PREFIX}``. These references will be
   overwritten with a placeholder that will be updated upon installation
   with the reference to the user's conda environment. Then they will be
   packaged together, together with the recipe, in a ``tar.gz`` file.
2. The build environment is then destroyed, and a new test environment
   is created. The test environment contains the dependencies listed in
   the ``run`` part of the ``meta.yaml`` file, plus optionally new ones
   listed in the ``test`` section in the ``meta.yaml``. It also
   obviously contains everything that ended up as part of your package
   at the end of the build step (i.e., contained in the ``tar.gz``
   file). The test environment is activated and the tests are run. This
   is meant to replicate a typical user situation, so you can only run
   here things that are part of the installation of the package (i.e.,
   gets installed with ``make install`` or similar during ``build.sh``.
   If instead the tests are part of the source package and are not
   installed during ``make install`` or similar, you will not be able to
   run them here. In this case you have two options: modify ``build.sh``
   so that they get installed (but it means that they will be installed
   for your users as well, which is not usually a good idea), or you
   should run these tests as part of the build process and reserve for
   this moment only functionality tests that the user is supposed to be
   able to run on its own.
3. Upon completion, you can optionally instruct ``conda build`` to
   upload the products to a channel of your choice, or you can do it
   manually later.

See here for a more in-depth description:
https://conda.io/docs/user-guide/tasks/build-packages/recipe.html

Release a package on conda-forge
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Building for conda-forge has a specific set of rules that are necessary
in order to guarantee the compatibility of your package with the other
packages in ``conda-forge``. See here for the appropriate documentation:
https://conda-forge.org/docs/.

