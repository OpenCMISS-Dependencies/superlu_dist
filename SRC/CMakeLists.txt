SET(ALLAUX sp_ienv.c etree.c sp_colorder.c get_perm_c.c
    mmd.c comm.c memory.c util.c superlu_grid.c
	pxerbla.c superlu_timer.c GetDiagU.c symbfact.c
	psymbfact.c psymbfact_util.c get_perm_c_parmetis.c mc64ad_dist.c
	static_schedule.c)

#### LAPACK auxiliary routines	
SET(LAAUX lsame.c xerbla.c slamch.c dlamch.c)
# Do not optimize these routines #
set_source_files_properties(${LAAUX} PROPERTIES COMPILE_FLAGS -O0)

SET(DSLUSRC	dlangs.c dgsequ.c dlaqgs.c dutil.c
	dmemory.c dmyblas2.c dsp_blas2.c dsp_blas3.c)
SET(ZSLUSRC	dcomplex.c zlangs.c zgsequ.c zlaqgs.c zutil.c
	  zmemory.c zmyblas2.c zsp_blas2.c zsp_blas3.c)

#
# Routines for double precision parallel SuperLU	  
SET(DPLUSRC pdgssvx.c pdgssvx_ABglobal.c dreadhb.c dreadrb.c dreadtriple.c dreadMM.c
	  pdgsequ.c pdlaqgs.c dldperm_dist.c pdlangs.c pdutil.c
	  pdsymbfact_distdata.c ddistribute.c pddistribute.c
	  pdgstrf.c pdgstrf2.c
	  pdgstrs.c pdgstrs1.c pdgstrs_lsum.c pdgstrs_Bglobal.c
	  pdgsrfs.c pdgsmv.c pdgsrfs_ABXglobal.c pdgsmv_AXglobal.c) 

#
# Routines for double complex parallel SuperLU
SET(ZPLUSRC pzgssvx.c pzgssvx_ABglobal.c zreadhb.c zreadrb.c zreadtriple.c zreadMM.c
	  pzgsequ.c pzlaqgs.c zldperm_dist.c pzlangs.c pzutil.c
	  pzsymbfact_distdata.c zdistribute.c pzdistribute.c
	  pzgstrf.c pzgstrf2.c
	  pzgstrs.c pzgstrs1.c pzgstrs_lsum.c pzgstrs_Bglobal.c
	  pzgsrfs.c pzgsmv.c pzgsrfs_ABXglobal.c pzgsmv_AXglobal.c)
	  
SET(SRC_DOUBLE ${ALLAUX} ${LAAUX} ${DSLUSRC} ${DPLUSRC})
SET(SRC_COMPLEX16 ${ALLAUX} ${LAAUX} ${ZSLUSRC} ${ZPLUSRC})

add_library(superlu_dist STATIC ${SRC_DOUBLE})
#target_compile_options(superlu_dist )
target_compile_definitions(superlu_dist PUBLIC 
    USE_VENDOR_BLAS # using ocms blas
    NoChange  # no change for fortran linking
    DEBUGlevel=0 PRNTlevel=1 PROFlevel=0 # misc
    _LONGINT # build type
)

target_link_libraries(superlu_dist 
    PUBLIC blas parmetis metis 
           ${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES}
)
target_include_directories(superlu_dist 
    PUBLIC SRC ${MPI_C_INCLUDE_PATH} ${MPI_Fortran_INCLUDE_PATH}
)
set_target_properties(superlu_dist PROPERTIES OUTPUT_NAME superlu-dist-${SUPERLU_DIST_VERSION})

export(TARGETS superlu_dist FILE ${OPENCMISS_DEPENDENCIES_CONFIGS_DIR}/superlu_dist-${SUPERLU_DIST_VERSION}/superlu-config.cmake)
include(CMakePackageConfigHelpers)
WRITE_BASIC_PACKAGE_VERSION_FILE(${OPENCMISS_DEPENDENCIES_CONFIGS_DIR}/superlu_dist-${SUPERLU_DIST_VERSION}/superlu-config-version.cmake
    VERSION ${SUPERLU_DIST_VERSION} COMPATIBILITY AnyNewerVersion)