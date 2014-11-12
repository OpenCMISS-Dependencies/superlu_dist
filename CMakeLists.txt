cmake_minimum_required (VERSION 3.0)
project (SUPERLU_DIST VERSION 4.3 LANGUAGES C Fortran)

option(BUILD_TESTS "${PROJECT_NAME} - Build tests" ON)
set(BUILD_PRECISION dz CACHE STRING "${PROJECT_NAME} - Build precisions (d|z|dz)")
option(USE_PARMETIS "Use ParMETIS library" ON)
#option(USE_METIS "Use METIS library" OFF)

find_package(BLAS CONFIG REQUIRED)
if (USE_PARMETIS)
    find_package(PARMETIS CONFIG REQUIRED)
#elseif(USE_METIS)
#    find_package(METIS 5 CONFIG REQUIRED)
endif()

find_package(MPI REQUIRED)

SET(CMAKE_C_COMPILER ${MPI_C_COMPILER})
SET(CMAKE_Fortran_COMPILER ${MPI_Fortran_COMPILER})
SET(CMAKE_C_FLAGS "-std=c99 -openmp")
SET(CMAKE_C_FLAGS_RELEASE "-O3") # not recognized: -w2 -mkl
SET(CMAKE_C_FLAGS_DEBUG "-O0 -Wall")
SET(CMAKE_Fortran_FLAGS "-Mnomain")
SET(CMAKE_Fortran_FLAGS_RELEASE "-fast")
#SET(CMAKE_Fortran_FLAGS_DEBUG )

# collecting source file defs in subdirectory
add_subdirectory(SRC)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory(EXAMPLE)
endif()