#
#  This is a sample CMakeLists.txt suitable for direct use with a new PETSc application or
#  you can add fragments of material below to an existing applications CMakeLists.txt
#
#  Usage:  To build ex1 from ex1.c
#    rm -fr build
#    mkdir build
#    cd build
#    cmake ..
#    make
#    ./ex1
#
#  By default it gets both the compiler information and the library information from PETSc
#
include(CMakePrintHelpers)
cmake_minimum_required(VERSION 3.1.0)

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(BLA_VENDOR Intel10_64lp)

message("before petsc pkg config")

# set root of location to find PETSc's pkg-config
set(PETSC $ENV{PETSC_DIR}/$ENV{PETSC_ARCH})
set(ENV{PKG_CONFIG_PATH} ${PETSC}/lib/pkgconfig)

message("after petsc pkg config")
message(${PETSC})

# Remove the lines below if you do not wish to have PETSc determine the compilers
execute_process ( COMMAND pkg-config PETSc --variable=ccompiler COMMAND tr -d '\n' OUTPUT_VARIABLE C_COMPILER)
SET(CMAKE_C_COMPILER ${C_COMPILER})
execute_process ( COMMAND pkg-config PETSc --variable=cxxcompiler COMMAND tr -d '\n' OUTPUT_VARIABLE CXX_COMPILER)
if (CXX_COMPILER)
  SET(CMAKE_CXX_COMPILER ${CXX_COMPILER})
  SET(CMAKE_CXX_LINK_EXECUTABLE "${CMAKE_CXX_LINK_EXECUTABLE} -ldl")
endif (CXX_COMPILER)
execute_process ( COMMAND pkg-config PETSc --variable=fcompiler COMMAND tr -d '\n' OUTPUT_VARIABLE FORTRAN_COMPILER)
cmake_print_variables(FORTRAN_COMPILER)

if (FORTRAN_COMPILER)
  message(${FORTRAN_COMPILER})
  SET(CMAKE_Fortran_COMPILER ${FORTRAN_COMPILER})
  # set(CMAKE_Fortran_FLAGS "${CMAKE_Fortran_FLAGS} -I${PETSC_DIR}/include -I${PETSC_DIR}/${PETSC_ARCH}/include")
  add_compile_options(-ffree-line-length-none -fallow-argument-mismatch -freal-4-real-8 -O3)
  add_link_options(-ldl -lstdc++ -Wl,--copy-dt-needed-entries)
  enable_language(Fortran C CXX)

  # SET(CMAKE_Fortran_LINK_EXECUTABLE "${CMAKE_Fortran_LINK_EXECUTABLE} -ldl")
endif (FORTRAN_COMPILER)


#  tells CMake to build the application ex1 from the source file ex1.c
#  this must appear AFTER the compilers are set
project(myexe)
add_executable(myexe exl95.F90)

find_package(PkgConfig REQUIRED)
find_package(LAPACK REQUIRED)

pkg_search_module(PETSC REQUIRED IMPORTED_TARGET PETSc)

# target_link_libraries(myexe LAPACK::LAPACK)

target_link_libraries(myexe LAPACK::LAPACK PkgConfig::PETSC)

if (PETSC_FOUND)
  message("PETSc found")
endif(PETSC_FOUND)
if (LAPACK_FOUND)
  message(STATUS "LAPACK libraries: ${LAPACK_LIBRARIES}")
endif(LAPACK_FOUND)
