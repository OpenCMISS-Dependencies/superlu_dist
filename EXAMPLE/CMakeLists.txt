macro(LINK TARGET)
    target_link_libraries(${TARGET} 
        PRIVATE superlu_dist blas)
    target_include_directories(${TARGET} PRIVATE ../SRC)
    if (UNIX)
        target_link_libraries(${TARGET} PRIVATE m)
    endif()
    add_test(${TARGET} ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${NF} ./${TARGET} ${TESTMATRIX})
endmacro()

FILE(COPY g20.rua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
FILE(COPY big.rua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
FILE(COPY cg20.cua DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

if(BUILD_PRECISION MATCHES [d])
    set(TESTMATRIX g20.rua)
    set(NF 2)
    add_executable(pddrive pddrive.c dcreate_matrix.c)
    LINK(pddrive)
    
    add_executable(pddrive1 pddrive1.c dcreate_matrix.c)
    LINK(pddrive1)
    
    add_executable(pddrive2 pddrive2.c dcreate_matrix.c)
    LINK(pddrive2)
    
    add_executable(pddrive3 pddrive3.c dcreate_matrix.c)
    LINK(pddrive3)
    
    
    
    add_executable(pddrive_ABglobal pddrive_ABglobal.c)
    LINK(pddrive_ABglobal)
    
    add_executable(pddrive_ABglobal1 pddrive1_ABglobal.c sp_ienv.c)
    LINK(pddrive_ABglobal1)
    
    add_executable(pddrive_ABglobal2 pddrive2_ABglobal.c)
    LINK(pddrive_ABglobal2)
    
    add_executable(pddrive_ABglobal3 pddrive3_ABglobal.c)
    LINK(pddrive_ABglobal3)
    
    # Those tests need at least 10 procs
    set(NF 10)
    add_executable(pddrive4 pddrive4.c dcreate_matrix.c)
    LINK(pddrive4)
    
    add_executable(pddrive_ABglobal4 pddrive4_ABglobal.c)
    LINK(pddrive_ABglobal4)
endif()

if(BUILD_PRECISION MATCHES [z])
    set(TESTMATRIX cg20.cua)
    set(NF 2)
    
    add_executable(pzdrive pzdrive.c zcreate_matrix.c sp_ienv.c)
    LINK(pzdrive)
    
    add_executable(pzdrive1 pzdrive1.c zcreate_matrix.c)
    LINK(pzdrive1)
    
    add_executable(pzdrive2 pzdrive2.c zcreate_matrix.c)
    LINK(pzdrive2)
    
    add_executable(pzdrive3 pzdrive3.c zcreate_matrix.c)
    LINK(pzdrive3)
    
    add_executable(pzdrive_ABglobal pzdrive_ABglobal.c)
    LINK(pzdrive_ABglobal)
    
    add_executable(pzdrive_ABglobal1 pzdrive1_ABglobal.c sp_ienv.c)
    LINK(pzdrive_ABglobal1)
    
    add_executable(pzdrive_ABglobal2 pzdrive2_ABglobal.c)
    LINK(pzdrive_ABglobal2)
    
    add_executable(pzdrive_ABglobal3 pzdrive3_ABglobal.c)
    LINK(pzdrive_ABglobal3)
    
    # Those tests need at least 10 procs
    set(NF 10)
    add_executable(pzdrive4 pzdrive4.c zcreate_matrix.c)
    LINK(pzdrive4)
    add_executable(pzdrive_ABglobal4 pzdrive4_ABglobal.c)
    LINK(pzdrive_ABglobal4)
endif()

