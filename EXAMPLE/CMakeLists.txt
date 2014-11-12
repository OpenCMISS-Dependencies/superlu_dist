macro(LINK TARGET)
    target_link_libraries(${TARGET} 
        PRIVATE superlu_dist blas)
    #if (USE_PARMETIS)
    #    target_link_libraries(${TARGET} PUBLIC parmetis metis)
    #endif()
                #${MPI_C_LIBRARIES} ${MPI_Fortran_LIBRARIES})
    target_include_directories(${TARGET} PRIVATE ../SRC)
    if (UNIX)
        target_link_libraries(${TARGET} PRIVATE m)
    endif()
    #target_compile_definitions(${TARGET} PUBLIC Add_)
endmacro()

add_executable(pddrive pddrive.c dcreate_matrix.c)
LINK(pddrive)

add_executable(pddrive1 pddrive1.c dcreate_matrix.c)
LINK(pddrive1)

add_executable(pddrive2 pddrive2.c dcreate_matrix.c)
LINK(pddrive2)

add_executable(pddrive3 pddrive3.c dcreate_matrix.c)
LINK(pddrive3)

add_executable(pddrive4 pddrive4.c dcreate_matrix.c)
LINK(pddrive4)

add_executable(pddrive_ABglobal pddrive_ABglobal.c)
LINK(pddrive_ABglobal)

add_executable(pddrive_ABglobal1 pddrive1_ABglobal.c sp_ienv.c)
LINK(pddrive_ABglobal1)

add_executable(pddrive_ABglobal2 pddrive2_ABglobal.c)
LINK(pddrive_ABglobal2)

add_executable(pddrive_ABglobal3 pddrive3_ABglobal.c)
LINK(pddrive_ABglobal3)

add_executable(pddrive_ABglobal4 pddrive4_ABglobal.c)
LINK(pddrive_ABglobal4)

FILE(COPY g20.rua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
FILE(COPY big.rua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
FILE(COPY cg20.cua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
