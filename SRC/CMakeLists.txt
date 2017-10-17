SET(ALLAUX sp_ienv.c etree.c sp_colorder.c get_perm_c.c
    mmd.c comm.c memory.c util.c superlu_grid.c
	pxerbla.c superlu_timer.c GetDiagU.c mc64ad.c symbfact.c
	psymbfact.c psymbfact_util.c
	# New in 4.1
	mc64ad_dist.c static_schedule.c)
if (USE_PARMETIS)
    LIST(APPEND ALLAUX get_perm_c_parmetis.c)
endif()

#### LAPACK auxiliary routines	
SET(LAAUX lsame.c xerbla.c slamch.c dlamch.c)
# Do not optimize these routines #
set_source_files_properties(${LAAUX} PROPERTIES COMPILE_FLAGS -O0)

SET(dSLUSRC	dlangs.c dgsequ.c dlaqgs.c dutil.c
	dmemory.c dmyblas2.c dsp_blas2.c dsp_blas3.c)
SET(zSLUSRC	dcomplex.c zlangs.c zgsequ.c zlaqgs.c zutil.c
	  zmemory.c zmyblas2.c zsp_blas2.c zsp_blas3.c)

# Routines for double precision parallel SuperLU	  
SET(dPLUSRC dldperm.c ddistribute.c pdgstrf.c pdgstrs_Bglobal.c
	  pdgstrs1.c pdgssvx_ABglobal.c pdgsrfs_ABXglobal.c pdgsmv_AXglobal.c
	  pdgssvx.c pdgstrs.c pddistribute.c pdlangs.c pdutil.c
	  pdgsequ.c pdlaqgs.c pdgsrfs.c pdgsmv.c pdgstrs_lsum.c
	  pdsymbfact_distdata.c dreadhb.c dreadrb.c dreadtriple.c
	  # New in 4.1
	  dldperm_dist.c dreadMM.c pdgstrf2.c)#

#
# Routines for double complex parallel SuperLU
SET(zPLUSRC zldperm.c zdistribute.c pzgssvx_ABglobal.c pzgstrf.c
	  pzgstrs1.c pzgstrs_Bglobal.c pzgsrfs_ABXglobal.c pzgsmv_AXglobal.c
	  pzgssvx.c pzgstrs.c pzdistribute.c pzlangs.c pzutil.c
	  pzgsequ.c pzlaqgs.c pzgsrfs.c pzgsmv.c pzgstrs_lsum.c
	  pzsymbfact_distdata.c zreadhb.c zreadtriple.c
	  # New in 4.1
	  zldperm_dist.c zreadMM.c pzgstrf2.c)
	  
SET(SRC_d ${dSLUSRC} ${dPLUSRC})
SET(SRC_z ${zSLUSRC} ${zPLUSRC})

SET(COMPDEFS
        USE_VENDOR_BLAS # using ocms blas
        ${FORTRAN_MANGLING}  # Fortran mangling scheme
        DEBUGlevel=0 PRNTlevel=1 PROFlevel=0 # misc
        #_LONGINT # This flag caused segfaults
)
add_library(auxiliary OBJECT ${ALLAUX} ${LAAUX})
target_compile_definitions(auxiliary PRIVATE ${COMPDEFS})
target_include_directories(auxiliary PRIVATE
         #$<$<BOOL:${USE_METIS}>:$<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
         $<$<BOOL:${USE_PARMETIS}>:
             $<TARGET_PROPERTY:parmetis,INTERFACE_INCLUDE_DIRECTORIES>
             $<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
    )
SET(OBJS $<TARGET_OBJECTS:auxiliary>)
if(BUILD_PRECISION MATCHES [d])
    add_library(slud_d OBJECT ${SRC_d})
    target_compile_definitions(slud_d PUBLIC ${COMPDEFS})
    target_include_directories(slud_d PRIVATE
         #$<$<BOOL:${USE_METIS}>:$<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
         $<$<BOOL:${USE_PARMETIS}>:
             $<TARGET_PROPERTY:parmetis,INTERFACE_INCLUDE_DIRECTORIES>
             $<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
    )
    LIST(APPEND OBJS "$<TARGET_OBJECTS:slud_d>")
endif()
if(BUILD_PRECISION MATCHES [z])
    add_library(slud_z OBJECT ${SRC_z})
    target_compile_definitions(slud_z PUBLIC ${COMPDEFS})
    target_include_directories(slud_z PRIVATE
         #$<$<BOOL:${USE_METIS}>:$<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
         $<$<BOOL:${USE_PARMETIS}>:
             $<TARGET_PROPERTY:parmetis,INTERFACE_INCLUDE_DIRECTORIES>
             $<TARGET_PROPERTY:metis,INTERFACE_INCLUDE_DIRECTORIES>>
    )
    LIST(APPEND OBJS "$<TARGET_OBJECTS:slud_z>")
endif()

add_library(superlu_dist ${OBJS})
target_compile_definitions(superlu_dist PUBLIC $<BUILD_INTERFACE:${COMPDEFS}>)
target_link_libraries(superlu_dist 
    PUBLIC $<BUILD_INTERFACE:blas>
    mpi-c
)
if (USE_PARMETIS)
    target_link_libraries(superlu_dist PUBLIC parmetis metis)
#elseif (USE_METIS)
#    target_link_libraries(superlu_dist PUBLIC metis)
endif()
target_include_directories(superlu_dist PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)
set_target_properties(superlu_dist PROPERTIES
    OUTPUT_NAME superlu-dist-${SUPERLU_DIST_VERSION})

install(TARGETS superlu_dist EXPORT superlu_dist-config
    DESTINATION lib
    INCLUDES DESTINATION include/superlu_dist)
install(EXPORT superlu_dist-config DESTINATION ${PACKAGE_CONFIG_DIR})
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/ DESTINATION include/superlu_dist
    FILES_MATCHING PATTERN "*.h")
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake 
    "include(CMakeFindDependencyMacro)\r\n"
    	"set(${PROJECT_NAME}_IMPORT_PREFIX \${_IMPORT_PREFIX})\r\n"
    "find_dependency(BLAS ${BLAS_VERSION})\r\n"
)
if (USE_PARMETIS)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake "find_dependency(PARMETIS ${PARMETIS_VERSION})\r\n")
endif()
if (USE_METIS)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake "find_dependency(METIS ${METIS_VERSION})\r\n")
endif()
if (WITH_OPENMP)
    file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake "find_dependency(OpenMP)\r\n")
endif()
file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake "set(_IMPORT_PREFIX \${${PROJECT_NAME}_IMPORT_PREFIX})")
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-version.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-dependencies.cmake
    DESTINATION ${PACKAGE_CONFIG_DIR})
