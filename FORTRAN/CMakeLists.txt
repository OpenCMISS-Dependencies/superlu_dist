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
    add_executable(f_pddrive dhbcode1.f90 f_pddrive.f90
        dcreate_dist_matrix.c superlu_c2f_dwrap.c
        $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_pddrive superlu_dist)
    
    add_executable(f_5x5 f_5x5.f90 sp_ienv.c
        dcreate_dist_matrix.c superlu_c2f_dwrap.c
        $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_5x5 superlu_dist)
    #ADDTEST(f77_main f77_main -h g20.rua)
endif()
if(BUILD_PRECISION MATCHES [z])
    add_executable(f_pzdrive zhbcode1.f90 f_pzdrive.f90
        zcreate_dist_matrix.c superlu_c2f_zwrap.c
        $<TARGET_OBJECTS:fmod>)
    target_link_libraries(f_pzdrive superlu_dist)
    #ADDTEST(f77_main f77_main -h cg20.rua)
endif()