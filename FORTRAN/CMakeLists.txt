macro(ADDTEST name exec arg matfile)
    add_test(NAME ${name}
        COMMAND ${CMAKE_COMMAND}
            -DCMD=$<TARGET_FILE:${exec}>
            -DARGS:STRING=${arg}
            -DINFILE=${CMAKE_CURRENT_SOURCE_DIR}/../EXAMPLE/${matfile}
            -P ${SUPERLU_SOURCE_DIR}/testhelper.cmake)
endmacro()

add_library(fmod OBJECT superlupara.f90 superlu_mod.f90)

SET(src-F )
if(BUILD_PRECISION MATCHES [d])
    add_library(f_pddrive_c dcreate_dist_matrix.c superlu_c2f_dwrap.c)
    target_link_libraries(f_pddrive_c superlu_dist)
    add_executable(f_pddrive dhbcode1.f90 f_pddrive.f90 $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_pddrive f_pddrive_c superlu_dist)
    add_test(f_pddrive ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} 4 f_pddrive)

    add_library(f_5x5_c sp_ienv.c dcreate_dist_matrix.c superlu_c2f_dwrap.c)
    target_link_libraries(f_5x5_c superlu_dist)
    add_executable(f_5x5 f_5x5.f90 $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_5x5 f_5x5_c superlu_dist)
    add_test(f_5x5 ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} 2 f_5x5)
endif()
if(BUILD_PRECISION MATCHES [z])
    add_library(f_pzdrive_c zcreate_dist_matrix.c superlu_c2f_zwrap.c)
    target_link_libraries(f_pzdrive_c superlu_dist)
    add_executable(f_pzdrive zhbcode1.f90 f_pzdrive.f90 $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_pzdrive f_pzdrive_c superlu_dist)
    add_test(f_pzdrive ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} 4 f_pzdrive)
endif()