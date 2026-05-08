cmake_minimum_required(VERSION 3.18)
include(ExternalProject)

project(MPACT_TPLs
    VERSION 0.1.0
    DESCRIPTION "Downloads and installs Official TPLs for MPACT"
    LANGUAGES C CXX Fortran
)

# TPL Version and SHA1 info
set(  MPICH_VERSION 4.2.3)
set(   HDF5_VERSION 1.10.1)
set( LAPACK_VERSION 3.7.1)
set(  PETSC_VERSION 3.13.6)
set(  SLEPC_VERSION 3.13.4)
set(    PNG_VERSION 1.6.55)
set(PUGIXML_VERSION 1.15)
set(   GMSH_VERSION 4.12.2)

set(    MPICH${MPICH_VERSION}_SHA1 1be671c6f1293ab7f4dbf7ac71c3890de3dabff0)
set(      HDF5${HDF5_VERSION}_SHA1 73b77a23ca099ac47d8241f633bf67430007c430)
set(  LAPACK${LAPACK_VERSION}_SHA1 84c4f7163b52b1bf1f6ca2193f6f48ed3dec0fab)
set(    PETSC${PETSC_VERSION}_SHA1 2b6475e092356f4cb67afccb755a853db6eb5285)
set(    SLEPC${SLEPC_VERSION}_SHA1 2dbc73a17adc87a36d52c991bcbfa81db036454b)
set(        PNG${PNG_VERSION}_SHA1 70b9c9c450f19cd50e686e977dfd709aecc09ac3)
set(PUGIXML${PUGIXML_VERSION}_SHA1 e52292c0cbf1c22f47dc4ff1dfd4a482071d1c9c)
set(      GMSH${GMSH_VERSION}_SHA1 e606e92d59b8bae245e5229f4bd543991d9fd5d7)

# Project CMake potions
option(${PROJECT_NAME}_INSTALL_MPI     "Installs MPICH-${MPICH_VERSION}"        ON)
option(${PROJECT_NAME}_INSTALL_HDF5    "Installs HDF5-${HDF5_VERSION}"          ON)
option(${PROJECT_NAME}_INSTALL_LAPACK  "Installs LAPACK-${LAPACK_VERSION}"      ON)
option(${PROJECT_NAME}_INSTALL_PETSC   "Installs PETSC-${PETSC_VERSION}"        ON)
option(${PROJECT_NAME}_INSTALL_SLEPC   "Installs SLEPC-${SLEPC_VERSION}"        ON)
option(${PROJECT_NAME}_INSTALL_PNG     "Installs LIBPNG-${PNG_VERSION}"         OFF)
option(${PROJECT_NAME}_INSTALL_PUGIXML "Installs PUGIXML-${PUGIXML_VERSION}"    OFF)
option(${PROJECT_NAME}_INSTALL_GMSH    "Installs GMSH-${GMSH_VERSION}"          OFF)
option(${PROJECT_NAME}_LOG_TO_SCREEN  "Print configure/build output to screen" OFF)
set(${PROJECT_NAME}_BUILD_PARALLEL 4 CACHE STRING "Number of processors to use for building TPLs")

message(STATUS "${PROJECT_NAME} Configure options\n\n"
    "    ${PROJECT_NAME}_INSTALL_MPI     = ${${PROJECT_NAME}_INSTALL_MPI}\n"
    "    ${PROJECT_NAME}_INSTALL_HDF5    = ${${PROJECT_NAME}_INSTALL_HDF5}\n"
    "    ${PROJECT_NAME}_INSTALL_LAPACK  = ${${PROJECT_NAME}_INSTALL_LAPACK}\n"
    "    ${PROJECT_NAME}_INSTALL_PETSC   = ${${PROJECT_NAME}_INSTALL_PETSC}\n"
    "    ${PROJECT_NAME}_INSTALL_SLEPC   = ${${PROJECT_NAME}_INSTALL_SLEPC}\n"
    "    ${PROJECT_NAME}_INSTALL_PNG     = ${${PROJECT_NAME}_INSTALL_PNG}\n"
    "    ${PROJECT_NAME}_INSTALL_PUGIXML = ${${PROJECT_NAME}_INSTALL_PUGIXML}\n"
    "    ${PROJECT_NAME}_INSTALL_GMSH    = ${${PROJECT_NAME}_INSTALL_GMSH}\n"
    "    ${PROJECT_NAME}_LOG_TO_SCREEN   = ${${PROJECT_NAME}_LOG_TO_SCREEN}\n"
    "    ${PROJECT_NAME}_BUILD_PARALLEL  = ${${PROJECT_NAME}_BUILD_PARALLEL}\n"
    "    CMAKE_INSTALL_PREFIX       = ${CMAKE_INSTALL_PREFIX}\n"
)

if(NOT ${PROJECT_NAME}_LOG_TO_SCREEN)
    set(LOG_VAL 1)
else()
    set(LOG_VAL 0)
endif()

#Find python and required packages
find_package(Python3 REQUIRED COMPONENTS Interpreter)
set(REQUIRED_PYTHON_PACKAGES numpy matplotlib pandas h5py)

if(Python3_VERSION VERSION_GREATER_EQUAL 3.13)
    # This is required by petsc-3.13.6 build system and was deprecated as
    # standard package in Python 3.11 and removed in Python 3.13
    # For python version greater than 3.12, this can be installed with
    # conda -c conda-forge install -y standard-xdrlib
    set(REQUIRED_PYTHON_PACKAGES ${REQUIRED_PYTHON_PACKAGES} standard-xdrlib)
#else()
#    set(REQUIRED_PYTHON_PACKAGES ${REQUIRED_PYTHON_PACKAGES} xdrlib)
endif()

foreach(PYPACKAGE ${REQUIRED_PYTHON_PACKAGES})
    execute_process(
        COMMAND ${Python3_EXECUTABLE} -c
            "from importlib.metadata import version; print(version('${PYPACKAGE}'))"
        RESULT_VARIABLE IMPORT_RESULT
        OUTPUT_VARIABLE PYPACKAGE_VERSION
        OUTPUT_STRIP_TRAILING_WHITESPACE
        ERROR_QUIET
    )

    if(IMPORT_RESULT EQUAL 0)
        message(STATUS "  Required Python Package: ${PYPACKAGE} -- FOUND")
        message(STATUS "    ${PYPACKAGE}: ${PYPACKAGE_VERSION}")
    else()
        message(FATAL_ERROR "\n"
            "  Required Python Package: ${PYPACKAGE} -- NOT FOUND\n"
            "    To install, try running...\n"
            "    $ conda install -y ${PYPACKAGE}   # for miniconda\n"
            "       ...or\n"
            "    $ pip install --user ${PYPACKAGE} # for pip\n"
        )
    endif()
endforeach()

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/load_dev_env.sh.in"
    "${CMAKE_INSTALL_PREFIX}/load_dev_env.sh"
    @ONLY
)
message(STATUS "Wrote environment script ${CMAKE_INSTALL_PREFIX}/load_dev_env.sh")

if(${PROJECT_NAME}_INSTALL_MPI)
    ExternalProject_Add(mpich-${MPICH_VERSION}
        URL         https://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz
        URL_HASH    SHA1=${MPICH${MPICH_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/mpich-${MPICH_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/mpich-${MPICH_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        CONFIGURE_COMMAND <SOURCE_DIR>/configure
          --prefix=<INSTALL_DIR>
          --enable-shared
          --enable-debuginfo
          --enable-fast=yes
          --enable-g=dbg
          CC=${CMAKE_C_COMPILER}
          FC=${CMAKE_Fortran_COMPILER}
          CXX=${CMAKE_CXX_COMPILER}
        BUILD_COMMAND   make -j${${PROJECT_NAME}_BUILD_PARALLEL}
        INSTALL_COMMAND make install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )

    ExternalProject_Get_Property(mpich-${MPICH_VERSION} INSTALL_DIR)
    set(MPI_ROOT ${INSTALL_DIR})
else()
    message(STATUS "Skipping download of MPICH-${MPICH_VERSION} because ${PROJECT_NAME}_INSTALL_MPI=${${PROJECT_NAME}_INSTALL_MPI}")
    set(MPI_ROOT "$ENV{MPI_ROOT}")
endif()


if(${PROJECT_NAME}_INSTALL_HDF5)
    ExternalProject_Add(hdf5-${HDF5_VERSION}
        DEPENDS     mpich-${MPICH_VERSION}
        URL         https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-${HDF5_VERSION}/src/hdf5-${HDF5_VERSION}.tar.gz
        URL_HASH    SHA1=${HDF5${HDF5_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/hdf5-${HDF5_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/hdf5-${HDF5_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        CONFIGURE_COMMAND <SOURCE_DIR>/configure
          --prefix=<INSTALL_DIR>
          --enable-fortran
          --enable-cxx
          --enable-hl
          --enable-parallel
          --enable-shared=yes
          --enable-symbols=yes
          --enable-build-mode=production
          --enable-unsupported
          CC=${MPI_ROOT}/bin/mpicc
          FC=${MPI_ROOT}/bin/mpifort
          CXX=${MPI_ROOT}/bin/mpicxx
        BUILD_COMMAND   make -j${${PROJECT_NAME}_BUILD_PARALLEL}
        INSTALL_COMMAND make install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )
else()
    message(STATUS "Skipping download of HDF5-${HDF5_VERSION} because ${PROJECT_NAME}_INSTALL_HDF5=${${PROJECT_NAME}_INSTALL_HDF5}")
endif()

if(${PROJECT_NAME}_INSTALL_LAPACK)
    ExternalProject_Add(lapack-${LAPACK_VERSION}
        URL         http://www.netlib.org/lapack/lapack-${LAPACK_VERSION}.tgz
        URL_HASH    SHA1=${LAPACK${LAPACK_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/lapack-${LAPACK_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/lapack-${LAPACK_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        CMAKE_ARGS
          -D CMAKE_Fortran_COMPILER=${CMAKE_Fortran_COMPILER}
          -D CMAKE_C_COMPILER=${CMAKE_C_COMPILER}
          -D CMAKE_BUILD_TYPE=Release
          -D CBLAS=ON
          -D LAPACKE=ON
          -D BUILD_SHARED_LIBS=ON
          -D CMAKE_INSTALL_PREFIX=<INSTALL_DIR>
          -D BUILD_TESTING=OFF
          -Wno-dev
        BUILD_COMMAND   make -j${${PROJECT_NAME}_BUILD_PARALLEL}
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )

    # PETSc looks for ${LAPACK_DIR}/lib64 so we create a lib64 symlink to lib
    ExternalProject_Add_Step(lapack-${LAPACK_VERSION} post_install
        COMMAND ${CMAKE_COMMAND} -E echo "       -- Running post-install step for lapack-${LAPACK_VERSION}"
        COMMAND ${CMAKE_COMMAND} -E create_symlink lib lib64
        WORKING_DIRECTORY <INSTALL_DIR>
        DEPENDEES install
        ALWAYS 1
    )
else()
    message(STATUS "Skipping download of LAPACK-${LAPACK_VERSION} because ${PROJECT_NAME}_INSTALL_LAPACK=${${PROJECT_NAME}_INSTALL_LAPACK}")
endif()

if(${PROJECT_NAME}_INSTALL_PETSC)
    ExternalProject_Add(petsc-${PETSC_VERSION}
        DEPENDS     lapack-${LAPACK_VERSION} mpich-4.2.3
        URL         https://web.cels.anl.gov/projects/petsc/download/release-snapshots/petsc-${PETSC_VERSION}.tar.gz
        URL_HASH    SHA1=${PETSC${PETSC_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/petsc-${PETSC_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/petsc-${PETSC_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        BUILD_IN_SOURCE            1
        CONFIGURE_COMMAND <SOURCE_DIR>/configure
            --prefix=<INSTALL_DIR>
            --with-cc=${MPI_ROOT}/bin/mpicc
            --with-fc=${MPI_ROOT}/bin/mpifort
            --with-cxx=${MPI_ROOT}/bin/mpicxx
            --with-cxx-dialect=C++11
            --with-fortran-bindings=1
            --with-shared-libraries=1
            --with-pic=1
            --with-debugging=0
            --with-blaslapack-dir=${CMAKE_INSTALL_PREFIX}/lapack-${LAPACK_VERSION}/lib64
            --with-x=0
            --with-batch=0
            --download-hypre=1
            --download-metis=1
            --download-parmetis=1
            --download-superlu_dist=1
            "--CFLAGS=-O3 -g"
            "--FFLAGS=-O3 -g"
            "--CXXFLAGS=-O3 -g"
        BUILD_COMMAND   make MAKE_NP=${${PROJECT_NAME}_BUILD_PARALLEL} PETSC_DIR=<SOURCE_DIR>
        INSTALL_COMMAND make PETSC_DIR=<SOURCE_DIR> install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )

    ExternalProject_Get_Property(petsc-${PETSC_VERSION} INSTALL_DIR)
    set(PETSC_ROOT ${INSTALL_DIR})
else()
   message(STATUS "Skipping download of petsc-${PETSC_VERSION} because ${PROJECT_NAME}_INSTALL_PETSC=${${PROJECT_NAME}_INSTALL_PETSC}")
   set(PETSC_ROOT "$ENV{PETSC_ROOT}")
endif()

if(${PROJECT_NAME}_INSTALL_SLEPC)
    ExternalProject_Add(slepc-${SLEPC_VERSION}
        DEPENDS     petsc-${PETSC_VERSION}
        URL         https://slepc.upv.es/download/distrib/slepc-${SLEPC_VERSION}.tar.gz
        URL_HASH    SHA1=${SLEPC${SLEPC_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/slepc-${SLEPC_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/slepc-${SLEPC_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        BUILD_IN_SOURCE            1
        CONFIGURE_COMMAND
          env PETSC_DIR=${PETSC_ROOT}
          <SOURCE_DIR>/configure
          --prefix=<INSTALL_DIR>
          --with-clean
        BUILD_COMMAND   make MAKE_NP=${${PROJECT_NAME}_BUILD_PARALLEL} SLEPC_DIR=<SOURCE_DIR> PETSC_DIR=${PETSC_ROOT}
        INSTALL_COMMAND make SLEPC_DIR=<SOURCE_DIR> PETSC_DIR=${PETSC_ROOT} install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )
else()
   message(STATUS "Skipping download of slepc-${SLEPC_VERSION} because ${PROJECT_NAME}_INSTALL_SLEPC=${${PROJECT_NAME}_INSTALL_SLEPC}")
endif()

if(${PROJECT_NAME}_INSTALL_PNG)
    ExternalProject_Add(libpng-${PNG_VERSION}
        URL         https://download.sourceforge.net/libpng/libpng-${PNG_VERSION}.tar.gz
        URL_HASH    SHA1=${PNG${PNG_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/libpng-${PNG_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/libpng-${PNG_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        BUILD_IN_SOURCE            1
        CONFIGURE_COMMAND
          <SOURCE_DIR>/configure
          --prefix=<INSTALL_DIR>
        BUILD_COMMAND   make -j${${PROJECT_NAME}_BUILD_PARALLEL}
        INSTALL_COMMAND make install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )
else()
   message(STATUS "Skipping download of libpng-${PNG_VERSION} because ${PROJECT_NAME}_INSTALL_PNG=${${PROJECT_NAME}_INSTALL_PNG}")
endif()

if(${PROJECT_NAME}_INSTALL_PUGIXML)
    ExternalProject_Add(pugixml-${PUGIXML_VERSION}
        URL         https://github.com/zeux/pugixml/archive/refs/tags/v${PUGIXML_VERSION}.tar.gz
        URL_HASH    SHA1=${PUGIXML${PUGIXML_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/pugixml-${PUGIXML_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/pugixml-${PUGIXML_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        CMAKE_ARGS
          -D CMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
          -D CMAKE_BUILD_TYPE=Release
          -D BUILD_SHARED_LIBS=ON
          -D CMAKE_INSTALL_PREFIX=<INSTALL_DIR>
        BUILD_COMMAND   make -j${${PROJECT_NAME}_BUILD_PARALLEL}
        INSTALL_COMMAND make install
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )
else()
   message(STATUS "Skipping download of pugixml-${PUGIXML_VERSION} because ${PROJECT_NAME}_INSTALL_PUGIXML=${${PROJECT_NAME}_INSTALL_PUGIXML}")
endif()

if(${PROJECT_NAME}_INSTALL_GMSH)
    ExternalProject_Add(gmsh-${GMSH_VERSION}
        URL         https://gmsh.info/bin/Linux/gmsh-${GMSH_VERSION}-Linux64-sdk.tgz
        URL_HASH    SHA1=${GMSH${GMSH_VERSION}_SHA1}
        PREFIX      ${CMAKE_BINARY_DIR}/external/gmsh-${GMSH_VERSION}
        INSTALL_DIR ${CMAKE_INSTALL_PREFIX}/gmsh-${GMSH_VERSION}
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        DOWNLOAD_NO_PROGRESS       ON
        CONFIGURE_COMMAND ""
        BUILD_COMMAND   ""
        INSTALL_COMMAND ${CMAKE_COMMAND} -E copy_directory <SOURCE_DIR> <INSTALL_DIR>
        LOG_CONFIGURE   ${LOG_VAL}
        LOG_BUILD       ${LOG_VAL}
        LOG_INSTALL     ${LOG_VAL}
    )
else()
   message(STATUS "Skipping download of gmsh-${GMSH_VERSION} because ${PROJECT_NAME}_INSTALL_GMSH=${${PROJECT_NAME}_INSTALL_GMSH}")
endif()
