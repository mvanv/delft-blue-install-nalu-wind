Building Nalu-Wind on Delft Blue
================================

Building Nalu-Wind requires several machine specific stept. Below, the steps required for installation on Delft Blue are presented.

Preparations
============

Before installation of any dependancies make sure that no unwanted modules are loaded that might confuse the dependencies (i.e. different versions of GCC):



    module purge

Then load the following modules:



    module load 2022r2
    module load openmpi
    module load cmake
    module load intel-mkl

Add the following directories to the PATHs to make sure the systems picks up the BLAS and LAPACK packages:



    export PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/bin:$PATH
    export CPATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/include:$CPATH
    export LIBRARY_PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/lib/intel64:$LIBRARY_PATH
    export LD_LIBRARY_PATH=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl/lib/intel64:$LD_LIBRARY_PATH

Building the dependencies
=========================

The following dependencies are already installed on Delft Blue: CMake, SuperLU, Libxml2, Boost, Zlib, HDF5, NetCDF, Parallel NetCDF. Trilinos and Paraview are also already installed, but require specific configuration. Therefore, we will build them ourselves.

YAML-cpp
--------

Build according to regular instructions

OpenFAST
--------

Clone using the regular instructions and configure with the following command to enable C++ API:

    cmake \
    -DBUILD_OPENFAST_CPP_API:BOOL=ON \
    -DBUILD_SHARED_LIBS:BOOL=ON \
    ..

Then
 
    make -j48 && make install

Trilinos
--------

Clone using the regular instruction and configure with the following ``do-config-trilinos`` script for Delft Blue.



    #!/bin/bash
    # The base directory where mpi is located.
    # From here you should be able to find include/mpi.h bin/mpicxx, bin/mpiexec, etc.
    MPI_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/openmpi-4.1.1-fezcq73heq4rzzsbcumuq5xx4v5asv45
    NALU_BUILD_DIR=/scratch/avanvondelen/10_Software/Nalu_Root
    # Note: Don't forget to set your LD_LIBRARY_PATH to $mpi_base_dir/lib
    # You may also need to add to LD_LIBRARY_PATH the lib directory for the compiler
    # used to create the mpi executables.
    # TPLS needed by trilinos, possibly provided by HomeBrew on a Mac
    BOOST_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/boost-1.77.0-gvqdfio5uj6a625h64y3jppv3jjfzx7m
    SUPERLU_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/superlu-5.3.0-2milooug6p4suldamkspcrrt7qkhs5jp
    NETCDF_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/netcdf-c-4.8.1-kz7m3osaphp3uut6i2tg5a5mdqf7q64m
    HDF5_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/hdf5-1.10.7-wscpmjfq75bppp3geu4xtecw3buxhnke
    PARALLEL_NETCDF_ROOT_DIR=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/parallel-netcdf-1.12.2-7vzruwzzdpsbb6alxc5v6pguq4smm6cc
    ZLIB_ROOT_DIR=/usr
    TRILINOS_ROOT_DIR=${NALU_BUILD_DIR}/install/trilinos
    MKL_ROOT=/apps/arch/2022r2/software/linux-rhel8-skylake_avx512/gcc-8.5.0/intel-mkl-2020.4.304-562z3j76h4zy26bjp5r2mrimud6fshrc/mkl
    EXTRA_ARGS=$@
    # Cleanup old cache before we configure
    # Note: This does not remove files produced by make. Use "make clean" for this.
    find . -name "CMakeFiles" -exec rm -rf {} \;
    rm -f CMakeCache.txt
    cmake \
    -DCMAKE_INSTALL_PREFIX=${TRILINOS_ROOT_DIR} \
    -DCMAKE_BUILD_TYPE:STRING=RELEASE \
    -DMPI_USE_COMPILER_WRAPPERS:BOOL=ON \
    -DMPI_CXX_COMPILER:FILEPATH=${CXX} \
    -DKokkos_ENABLE_DEPRECATED_CODE:BOOL=OFF \
    -DTpetra_INST_SERIAL:BOOL=ON \
    -DTrilinos_ENABLE_CXX11:BOOL=ON \
    -DTrilinos_ENABLE_EXPLICIT_INSTANTIATION:BOOL=ON \
    -DTpetra_INST_DOUBLE:BOOL=ON \
    -DTpetra_INST_COMPLEX_DOUBLE:BOOL=OFF \
    -DTrilinos_ENABLE_TESTS:BOOL=OFF \
    -DTrilinos_ENABLE_ALL_OPTIONAL_PACKAGES:BOOL=OFF \
    -DTrilinos_ASSERT_MISSING_PACKAGES:BOOL=OFF \
    -DTrilinos_ALLOW_NO_PACKAGES:BOOL=OFF \
    -DTrilinos_ENABLE_Epetra:BOOL=OFF \
    -DTrilinos_ENABLE_Tpetra:BOOL=ON \
    -DTrilinos_ENABLE_KokkosKernels:BOOL=ON \
    -DTrilinos_ENABLE_ML:BOOL=OFF \
    -DTrilinos_ENABLE_MueLu:BOOL=ON \
    -DXpetra_ENABLE_Kokkos_Refactor:BOOL=ON \
    -DMueLu_ENABLE_Kokkos_Refactor:BOOL=ON \
    -DTrilinos_ENABLE_EpetraExt:BOOL=OFF \
    -DTrilinos_ENABLE_AztecOO:BOOL=OFF \
    -DTrilinos_ENABLE_Belos:BOOL=ON \
    -DTrilinos_ENABLE_Ifpack2:BOOL=ON \
    -DTrilinos_ENABLE_Amesos2:BOOL=ON \
    -DTrilinos_ENABLE_Zoltan2:BOOL=ON \
    -DTrilinos_ENABLE_Ifpack:BOOL=OFF \
    -DTrilinos_ENABLE_Amesos:BOOL=OFF \
    -DTrilinos_ENABLE_Zoltan:BOOL=ON \
    -DTrilinos_ENABLE_STK:BOOL=ON \
    -DTrilinos_ENABLE_Gtest:BOOL=ON \
    -DTrilinos_ENABLE_SEACASExodus:BOOL=ON \
    -DTrilinos_ENABLE_SEACASEpu:BOOL=ON \
    -DTrilinos_ENABLE_SEACASExodiff:BOOL=ON \
    -DTrilinos_ENABLE_SEACASNemspread:BOOL=ON \
    -DTrilinos_ENABLE_SEACASNemslice:BOOL=ON \
    -DTrilinos_ENABLE_SEACASIoss:BOOL=ON \
    -DTPL_ENABLE_MPI:BOOL=ON \
    -DTPL_ENABLE_Boost:BOOL=ON \
    -DBoostLib_INCLUDE_DIRS:PATH=${BOOST_ROOT_DIR}/include \
    -DBoostLib_LIBRARY_DIRS:PATH=${BOOST_ROOT_DIR}/lib64 \
    -DBoost_INCLUDE_DIRS:PATH=${BOOST_ROOT_DIR}/include \
    -DBoost_LIBRARY_DIRS:PATH=${BOOST_ROOT_DIR}/lib64 \
    -DTPL_ENABLE_SuperLU:BOOL=ON \
    -DSuperLU_INCLUDE_DIRS:PATH=${SUPERLU_ROOT_DIR}/include \
    -DSuperLU_LIBRARY_DIRS:PATH=${SUPERLU_ROOT_DIR}/lib \
    -DTPL_ENABLE_Netcdf:BOOL=ON \
    -DNetCDF_ROOT:PATH=${NETCDF_ROOT_DIR} \
    -DTPL_Netcdf_PARALLEL:BOOL=ON \
    -DTPL_ENABLE_Pnetcdf:BOOL=ON \
    -DPNetCDF_ROOT:PATH=${PARALLEL_NETCDF_ROOT_DIR} \
    -DPnetcdf_INCLUDE_DIRS:PATH=${PARALLEL_NETCDF_ROOT_DIR}/include \
    -DPnetcdf_LIBRARY_DIRS:PATH=${PARALLEL_NETCDF_ROOT_DIR}/lib \
    -DTPL_ENABLE_HDF5:BOOL=ON \
    -DHDF5_ROOT:PATH=${HDF5_ROOT_DIR} \
    -DHDF5_NO_SYSTEM_PATHS:BOOL=ON \
    -DTPL_ENABLE_Zlib:BOOL=ON \
    -DZlib_INCLUDE_DIRS:PATH=${ZLIB_ROOT_DIR}/include \
    -DZlib_LIBRARY_DIRS:PATH=${ZLIB_ROOT_DIR}/lib \
    -DBLAS_LIBRARY_DIRS:FILEPATH="${MKLROOT}/lib/intel64" \
    -DBLAS_LIBRARY_NAMES:STRING="mkl_rt" \
    -DLAPACK_LIBRARY_DIRS:FILEPATH="${MKLROOT}/lib/intel64" \
    -DLAPACK_LIBRARY_NAMES:STRING="mkl_rt" \
    -DTPL_ENABLE_MKL:BOOL=ON \
    -DMKL_LIBRARY_DIRS:FILEPATH="${MKLROOT}/lib/intel64" \
    -DMKL_LIBRARY_NAMES:STRING="mkl_rt" \
    -DMKL_INCLUDE_DIRS:FILEPATH="${MKLROOT}/include" \
    $EXTRA_ARGS \
    ..

The CMakeFiles will try to link to the -lhdf5-shared library. This does not exist. A quick and dirty solution to solve this is to manually remove this link from all ``link.txt`` files containing this reference. These files can be found in the following directories in the build directory:



    find packages/seacas/*/*/CMakeFiles/*/ -name "link.txt"
    find packages/seacas/libraries/ioss/src/main/CMakeFiles/*/ -name "link.txt"

And they are present in the following directories:



    packages/stk/stk_tools/stk_tools/CMakeFiles/stk_block_extractor.dir
    packages/stk/stk_balance/stk_balance/CMakeFiles/*

HYPRE
-----

Build according to regular instructions

Paraview
--------

Not yet implemented

Nalu-Wind
---------

Clone according to regular instructions. Configure using ``do-config-nalu-wind_DelftBlue.sh``

	#!/bin/bash -l
	# Instructions:
	# Make a directory in the Nalu-Wind directory for building,
	# Copy this script to that directory and edit the
	# options below to your own needs and run it.
	NALU_ROOT_DIR=/scratch/avanvondelen/10_Software/Nalu_Root
	CXX_COMPILER=mpicxx
	C_COMPILER=mpicc
	FLAGS="-O2 -march=native -mtune=native"
	OVERSUBSCRIBE_FLAGS="--use-hwthread-cpus --oversubscribe"
	set -e
	TRILINOS_ROOT_DIR=${NALU_ROOT_DIR}/install/trilinos
	YAML_CPP_ROOT_DIR=${NALU_ROOT_DIR}/install/yaml-cpp
	# Clean before cmake configure
	set +e
	rm -rf CMakeFiles
	rm -f CMakeCache.txt
	set -e
	# Extra TPLs that can be included in the cmake configure:
	# -DENABLE_PARAVIEW_CATALYST:BOOL=ON \
	# -DPARAVIEW_CATALYST_INSTALL_PATH:PATH=${CATALYST_IOSS_ADAPTER_ROOT_DIR} \
	# -DENABLE_TIOGA:BOOL=ON \
	# -DTIOGA_DIR:PATH=${TIOGA_ROOT_DIR} \
	(set -x; cmake \
	-DENABLE_OPENFAST:BOOL=ON \
	-DOpenFAST_DIR:PATH=/home/avanvondelen/OpenFAST/install \
	-DENABLE_HYPRE:BOOL=ON \
	-DHYPRE_DIR:PATH=${NALU_ROOT_DIR}/install/hypre \
	-DCMAKE_CXX_COMPILER:STRING=${CXX_COMPILER} \
	-DCMAKE_CXX_FLAGS:STRING="${FLAGS}" \
	-DMPI_CXX_COMPILER:STRING=${CXX_COMPILER} \
	-DMPIEXEC_PREFLAGS:STRING="${OVERSUBSCRIBE_FLAGS}" \
	-DTrilinos_DIR:PATH=${TRILINOS_ROOT_DIR} \
	-DYAML_DIR:PATH=${YAML_CPP_ROOT_DIR} \
	-DCMAKE_BUILD_TYPE:STRING=RELEASE \
	-DENABLE_DOCUMENTATION:BOOL=OFF \
	-DENABLE_TESTS:BOOL=ON \
	..)


Again, here, the link to -lhdf5-shared needs to be removed manually from the ``link.txt`` files. They are in the following directories in the build directory



    CMakeFiles/unittestX.dir
    CMakeFiles/naluX.dir

Then build using



    make -j48


