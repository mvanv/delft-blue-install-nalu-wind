Building Nalu-Wind on Delft Blue
================================

Building Nalu-Wind requires several machine specific stept. Below, the steps required for installation on Delft Blue are presented.

Preparations
------------

Before installation of any dependancies make sure that no unwanted modules are loaded that might confuse the dependencies (i.e. different versions of GCC):

::

    module purge

Then load the following modules:

::

    module load 2022r2
    module load openmpi
    module load cmake
    module load intel-mkl

Add the following directories to the PATHs to make sure the systems picks up the BLAS and LAPACK packages:

::

    export PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/bin:$PATH
    export CPATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/include:$CPATH
    export LIBRARY_PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/lib/intel64:$LIBRARY_PATH
    export LD_LIBRARY_PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/lib/intel64:$LD_LIBRARY_PATH


Building the dependencies
-------------------------

The following dependencies are already installed on Delft Blue: CMake, SuperLU, Libxml2, Boost, Zlib, HDF5, NetCDF, Parallel NetCDF. Trilinos and Paraview are also already installed, but require specific configuration. Therefore, we will build them ourselves.

YAML-cpp
~~~~~~~~

Build according to regular instructions

Trilinos
~~~~~~~~

Clone using the regular instruction and configure with the following ``do-config-trilinos`` script for Delft Blue.

.. literalinclude:: do-config-trilinos_DelftBlue.sh

The CMakeFiles will try to link to the -lhdf5-shared library. This does not exist. A quick and dirty solution to solve this is to manually remove this link from all ``link.txt`` files containing this reference. These files can be found in the following directories in the build directory:

::

    find packages/seacas/*/*/CMakeFiles/*/ -name "link.txt"
    find packages/seacas/libraries/ioss/src/main/CMakeFiles/*/ -name "link.txt"

And they are present in the following directories:

::

    packages/stk/stk_tools/stk_tools/CMakeFiles/stk_block_extractor.dir
    packages/stk/stk_balance/stk_balance/CMakeFiles/*

HYPRE
~~~~~

Build according to regular instructions

Paraview
~~~~~~~~

Not yet implemented

Nalu-Wind
~~~~~~~~~

Clone according to regular instructions. Configure using ``do-config-nalu-wind_DelftBlue.sh``

.. literalinclude:: do-config-trilinos_DelftBlue.sh


Again, here, the link to -lhdf5-shared needs to be removed manually from the ``link.txt`` files. They are in the following directories in the build directory

::

    CMakeFiles/unittestX.dir
    CMakeFiles/naluX.dir

Then build using

::

    make -j48


