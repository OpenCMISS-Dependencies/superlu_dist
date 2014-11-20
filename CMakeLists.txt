cmake_minimum_required (VERSION 3.0)
project (SUPERLU_DIST VERSION 3.3 LANGUAGES C Fortran)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
set(BUILD_PRECISION dz CACHE STRING "${PROJECT_NAME} - Build precisions (d|z|dz)")
option(USE_PARMETIS "Use ParMETIS library (includes use of METIS)" ON)
option(USE_METIS "Use METIS library" OFF)

find_package(BLAS ${BLAS_VERSION} REQUIRED)
if (USE_PARMETIS)
    find_package(PARMETIS ${PARMETIS_VERSION} REQUIRED)
elseif(USE_METIS)
    find_package(METIS ${METIS_VERSION} REQUIRED)
endif()

find_package(MPI REQUIRED)

SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
SET(CMAKE_C_FLAGS "-std=c99 -openmp -Wno-format")
SET(CMAKE_C_FLAGS_RELEASE "-O3") # not recognized: -w2 -mkl
SET(CMAKE_C_FLAGS_DEBUG "-O0 -Wall")

# collecting source file defs in subdirectory
add_subdirectory(SRC)
add_subdirectory(FORTRAN)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(EXAMPLE)
endif()