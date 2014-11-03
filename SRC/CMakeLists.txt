SET(ALLAUX sp_ienv.c etree.c sp_colorder.c get_perm_c.c
    mmd.c comm.c memory.c util.c superlu_grid.c
	pxerbla.c superlu_timer.c GetDiagU.c symbfact.c
	psymbfact.c psymbfact_util.c mc64ad_dist.c
	static_schedule.c)
if (USE_PARMETIS)
    LIST(APPEND ALLAUX get_perm_c_parmetis.c)
    set_source_files_properties(get_perm_c_parmetis.c 
        PROPERTIES COMPILE_FLAGS -DPARMETIS)
endif()
if (USE_METIS)
    set_source_files_properties(get_perm_c.c
        PROPERTIES COMPILE_FLAGS -DMETIS)
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
SET(dPLUSRC pdgssvx.c pdgssvx_ABglobal.c dreadhb.c dreadrb.c dreadtriple.c dreadMM.c
	  pdgsequ.c pdlaqgs.c dldperm_dist.c pdlangs.c pdutil.c
	  pdsymbfact_distdata.c ddistribute.c pddistribute.c
	  pdgstrf.c pdgstrf2.c
	  pdgstrs.c pdgstrs1.c pdgstrs_lsum.c pdgstrs_Bglobal.c
	  pdgsrfs.c pdgsmv.c pdgsrfs_ABXglobal.c pdgsmv_AXglobal.c) 

#
# Routines for double complex parallel SuperLU
SET(zPLUSRC pzgssvx.c pzgssvx_ABglobal.c zreadhb.c zreadrb.c zreadtriple.c zreadMM.c
	  pzgsequ.c pzlaqgs.c zldperm_dist.c pzlangs.c pzutil.c
	  pzsymbfact_distdata.c zdistribute.c pzdistribute.c
	  pzgstrf.c pzgstrf2.c
	  pzgstrs.c pzgstrs1.c pzgstrs_lsum.c pzgstrs_Bglobal.c
	  pzgsrfs.c pzgsmv.c pzgsrfs_ABXglobal.c pzgsmv_AXglobal.c)
	  
SET(SRC_d ${ALLAUX} ${LAAUX} ${dSLUSRC} ${dPLUSRC})
SET(SRC_z ${ALLAUX} ${LAAUX} ${zSLUSRC} ${zPLUSRC})

SET(COMPDEFS
        USE_VENDOR_BLAS # using ocms blas
        Add_  # no change for fortran linking
        DEBUGlevel=0 PRNTlevel=1 PROFlevel=0 # misc
        _LONGINT # build type
)
SET(OBJS )
if(BUILD_PRECISION MATCHES [d])
    add_library(slud_d OBJECT ${SRC_d})
    target_compile_definitions(slud_d PUBLIC ${COMPDEFS})
    LIST(APPEND OBJS "$<TARGET_OBJECTS:slud_d>")
endif()
if(BUILD_PRECISION MATCHES [z])
    add_library(slud_z OBJECT ${SRC_z})
    target_compile_definitions(slud_d PUBLIC ${COMPDEFS})
    LIST(APPEND OBJS "$<TARGET_OBJECTS:slud_z>")
endif()

add_library(superlu_dist STATIC ${OBJS})

target_link_libraries(superlu_dist 
    PUBLIC blas
    #${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES}
)
if (USE_PARMETIS)
    target_link_libraries(superlu_dist PUBLIC parmetis metis)
elseif (USE_METIS)
    target_link_libraries(superlu_dist PUBLIC metis)
endif()
target_include_directories(superlu_dist 
    PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/SRC>
    #${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
)
set_target_properties(superlu_dist PROPERTIES
    OUTPUT_NAME superlu-dist-${SUPERLU_DIST_VERSION})

install(TARGETS superlu_dist EXPORT superlu_dist-config
    DESTINATION lib
    INCLUDES DESTINATION include/superlu_dist)
install(EXPORT superlu_dist-config
    DESTINATION lib/cmake)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/ DESTINATION include/superlu_dist
    FILES_MATCHING PATTERN "*.h")
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-version.cmake
    COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/superlu_dist-config-version.cmake
    DESTINATION lib/cmake)