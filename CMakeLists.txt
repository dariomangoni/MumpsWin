# MUMPS 5.0 CMake file
# - all UPPER case "libraries" compile .obj files without linking
# WARNING: Some native build systems may not like targets that have only object files, so consider adding at least one real source file to any target that references $<TARGET_OBJECTS:objlib>

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(MumpsBuild C Fortran)

#################### Mumps configuration ####################
# Arithmetic configuration
SET(ARITHMETIC "double" CACHE STRING "MUMPS arithmetic")
SET_PROPERTY(CACHE ARITHMETIC PROPERTY STRINGS float double complex complex-double)


IF (ARITHMETIC MATCHES "float")
	SET(ARITH_PREFIX "s")
ELSEIF (ARITHMETIC MATCHES "double")
	SET(ARITH_PREFIX "d")
ELSEIF (ARITHMETIC MATCHES "complex")
	SET(ARITH_PREFIX "c")
ELSEIF (ARITHMETIC MATCHES "complex-double")
	SET(ARITH_PREFIX "z")
ENDIF ()

MESSAGE(STATUS "Arithmetic set to '${ARITHMETIC}'. Added '${ARITH_PREFIX}' prefix...")

# Ordering libraries selection
SET(METIS_USE FALSE CACHE BOOL "Select if METIS ordering is used")

IF (METIS_USE)
	SET(METIS_DIR "${CMAKE_SOURCE_DIR}" CACHE PATH "Path to METIS folder")
ENDIF (METIS_USE)

# System configuration
SET(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install" CACHE PATH "MUMPS install prefix" FORCE)

IF(CMAKE_SIZEOF_VOID_P MATCHES 4)
    SET(TARGET_SYSTEM "32")
ELSE()
    SET(TARGET_SYSTEM "64")
ENDIF()

MESSAGE(STATUS "Building ${TARGET_SYSTEM}bit target")


#################### Compiler and linker flags ####################
# -O2 optimization flags are already set by generator
SET(GENERIC_COMPILER_FLAGS_FORTRAN #aka OPTF
			"-MD -Dintel_ -DALLOW_NON_INIT -fpp")
SET(GENERIC_COMPILER_FLAGS_C #aka OPTC
			"-MD")

SET(COMPILER_DEFS_C #aka CDEFS
			"Add_")
SET(COMPILER_DEFS_FORTRAN #aka CDEF
			"")
			
SET(ORDERINGS_COMPILER_FLAGS_FORTRAN #aka ORDERINGSF
			"-Dpord")
SET(ORDERINGS_COMPILER_FLAGS_C #aka ORDERINGSC (modified)
			"-Dpord -DMUMPS_ARITH=MUMPS_ARITH_${ARITH_PREFIX}")

IF (METIS_USE)
	SET(ORDERINGS_COMPILER_FLAGS_C "${ORDERINGS_COMPILER_FLAGS_C} -Dmetis" )
	SET(ORDERINGS_COMPILER_FLAGS_FORTRAN "${ORDERINGS_COMPILER_FLAGS_FORTRAN} -Dmetis" )
ENDIF()
	


#################### Generic&External include directories ##########################
IF(TARGET_SYSTEM MATCHES "32")
	LINK_DIRECTORIES("C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/ia32")
ELSE()
	LINK_DIRECTORIES("C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/intel64")
ENDIF()
	
			
SET(MUMPS_INCLUDE #aka INCS
			"${CMAKE_SOURCE_DIR}/include") #aka INCSEQ
			
SET(BLAS_INCLUDE #aka nothing
			"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/include"
			"C:/Program Files (x86)/Intel/Composer XE 2015/compiler/include")

IF (METIS_USE)
	SET(ORDERINGS_INCLUDE "${ORDERINGS_INCLUDE}" "${METIS_DIR}/include")
ENDIF (METIS_USE)	
			
#################### Generic&External headers ##########################
SET(MUMPS_HEADERS
			"${MUMPS_INCLUDE}/${ARITH_PREFIX}mumps_c.h"
			"${MUMPS_INCLUDE}/${ARITH_PREFIX}mumps_root.h"
			"${MUMPS_INCLUDE}/${ARITH_PREFIX}mumps_struc.h"
			"${MUMPS_INCLUDE}/mumps_c_types.h"
			"${MUMPS_INCLUDE}/mumps_compat.h")				
										
#################### Generic&External headers ##########################
SET(ORDERINGS_LIB #aka LORDERINGS
			"${CMAKE_SOURCE_DIR}/lib/libpord.lib") #aka LPORDDIR+LPORD

IF (METIS_USE)
	SET(ORDERINGS_LIB "${ORDERINGS_LIB}" "${METIS_DIR}/libmetis/$<CONFIG>/metis.lib") #aka LPORDDIR+LPORD
ENDIF (METIS_USE)

			
SET(LIBSEQ_LIB
			"${CMAKE_SOURCE_DIR}/libseq/libmpiseq.lib") #aka LIBSEQ
			
IF (TARGET_SYSTEM MATCHES "32")
	SET(BLAS_LIB #aka LIBBLAS
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/ia32/mkl_intel_c.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/ia32/mkl_intel_thread.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/ia32/mkl_core.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/ia32/libiomp5md.lib")
ELSEIF (TARGET_SYSTEM MATCHES "64")
	SET(BLAS_LIB #aka LIBBLAS
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_intel_lp64.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_intel_thread.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_core.lib"
				"C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/intel64/libiomp5md.lib")
ENDIF ()

#######################################################
#################### TARGETS ##########################
#######################################################

############### PORD ###############
SET(PORD_DIR "${CMAKE_SOURCE_DIR}/PORD")
SET(PORD_INCLUDE "${PORD_DIR}/include")
SET(PORD_SOURCES
		"${PORD_DIR}/lib/bucket.c"
		"${PORD_DIR}/lib/ddbisect.c"
		"${PORD_DIR}/lib/ddcreate.c"
		"${PORD_DIR}/lib/gbipart.c"
		"${PORD_DIR}/lib/gbisect.c"
		"${PORD_DIR}/lib/gelim.c"
		"${PORD_DIR}/lib/graph.c"
		"${PORD_DIR}/lib/interface.c"
		"${PORD_DIR}/lib/minpriority.c"
		"${PORD_DIR}/lib/multisector.c"
		"${PORD_DIR}/lib/nestdiss.c"
		"${PORD_DIR}/lib/sort.c"
		"${PORD_DIR}/lib/symbfac.c"
		"${PORD_DIR}/lib/tree.c")
		
SET(PORD_HEAD
		"${PORD_INCLUDE}/const.h"
		"${PORD_INCLUDE}/eval.h"
		"${PORD_INCLUDE}/macros.h"
		"${PORD_INCLUDE}/params.h"
		"${PORD_INCLUDE}/protos.h"
		"${PORD_INCLUDE}/space.h"
		"${PORD_INCLUDE}/types.h")


ADD_LIBRARY(libpord STATIC
						${PORD_SOURCES}
						${PORD_HEAD})

SET_TARGET_PROPERTIES(libpord 	PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_C}"
						LINKER_LANGUAGE C)
						
TARGET_INCLUDE_DIRECTORIES(libpord	PUBLIC	"${PORD_INCLUDE}")
SET(ORDERINGS_INCLUDE "${ORDERINGS_INCLUDE}" "${PORD_INCLUDE}")

############### libseq ###############
SET(LIBSEQ_SRC_C
		"libseq/mpic.c"
		"libseq/elapse.c")

SET(LIBSEQ_SRC_FORTRAN
		"libseq/mpi.f")
		
SET(LIBSEQ_INCLUDE
		"${CMAKE_SOURCE_DIR}/libseq") #aka INCSEQ	
	
SET(LIBSEQ_HEAD_C
		"${LIBSEQ_INCLUDE}/mpi.h"
		"${LIBSEQ_INCLUDE}/elapse.h")

SET(LIBSEQ_HEAD_FORTRAN
		"${LIBSEQ_INCLUDE}/mpif.h")

# libseq object files (Fortran)
ADD_LIBRARY(LIBMPISEQ_FORTRAN OBJECT
						${LIBSEQ_SRC_FORTRAN}
						${LIBSEQ_HEAD_FORTRAN})

SET_TARGET_PROPERTIES(LIBMPISEQ_FORTRAN PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_FORTRAN}"
						LINK_FLAGS "-c"
						LINKER_LANGUAGE FORTRAN)

# libseq object files (C)
ADD_LIBRARY(LIBMPISEQ_C OBJECT
						${LIBSEQ_SRC_C}
						${LIBSEQ_HEAD_C})

SET_TARGET_PROPERTIES(LIBMPISEQ_C PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_C}"
						LINK_FLAGS "-c"
						COMPILE_DEFINITIONS "${COMPILER_DEFS_C}"
						LINKER_LANGUAGE C)

TARGET_INCLUDE_DIRECTORIES(LIBMPISEQ_C PUBLIC "${LIBSEQ_INCLUDE}")

# libseq library
ADD_LIBRARY(libmpiseq STATIC
						${LIBSEQ_HEAD_C}
						${LIBSEQ_HEAD_FORTRAN}
						$<TARGET_OBJECTS:LIBMPISEQ_C>
						$<TARGET_OBJECTS:LIBMPISEQ_FORTRAN>)
						
SET_PROPERTY(TARGET libmpiseq PROPERTY ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/libseq")
SET_PROPERTY(TARGET libmpiseq PROPERTY ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}/libseq")
SET_PROPERTY(TARGET libmpiseq PROPERTY ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_SOURCE_DIR}/libseq")
								
############### MUMPS (src folder) ###############
# Common MUMPS files
SET (MUMPS_COMMON_SRC_FORTRAN
        src/ana_omp_m.F
		src/tools_common.F
		src/tools_common_mod.F
        src/mumps_static_mapping.F
        src/mumps_sol_es.F
		src/fac_future_niv2_mod.F
        src/mumps_comm_ibcast.F
		src/mumps_ooc_common.F
        src/double_linked_list.F
        src/fac_asm_build_sort_index_m.F
        src/fac_asm_build_sort_index_ELT_m.F
        src/omp_tps_common_m.F
        src/mumps_l0_omp_m.F
        src/front_data_mgt_m.F
        src/fac_maprow_data_m.F
        src/fac_descband_data_m.F
        src/fac_ibct_data_m.F)
		
SET (MUMPS_COMMON_OTHER_SRC_FORTRAN
        src/ana_orderings.F
        src/ana_AMDMF.F
        src/bcast_errors.F
        src/estim_flops.F
        src/mumps_type_size.F
        src/mumps_type2_blocking.F
        src/mumps_version.F
        src/tools_common.F
		src/tools_common_mod.F
		src/mumps_print_defined.F)
		
SET (MUMPS_COMMON_OTHER_SRC_C
		src/mumps_c.c
        src/mumps_common.c
        src/mumps_orderings.c
        src/mumps_size.c
        src/mumps_io.c
        src/mumps_io_basic.c
        src/mumps_io_thread.c
        src/mumps_io_err.c
        src/mumps_numa.c)
		
SET (MUMPS_COMMON_OTHER_HEAD
		src/mumps_common.h
		src/mumps_headers.h
		src/mumps_io.h
		src/mumps_io_basic.h
		src/mumps_io_err.h
		src/mumps_io_thread.h
		src/mumps_orderings.h
		src/mumps_size.h
		src/mumps_tags.h)
		


# Common MUMPS object files (Fortran)
ADD_LIBRARY(MUMPS_COMMON_OBJ_FORTRAN OBJECT
						${MUMPS_COMMON_OTHER_HEAD} # not sure IF needed
						${MUMPS_COMMON_SRC_FORTRAN}
						${MUMPS_COMMON_OTHER_SRC_FORTRAN})
				
SET_TARGET_PROPERTIES(MUMPS_COMMON_OBJ_FORTRAN PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_FORTRAN} ${ORDERINGS_COMPILER_FLAGS_FORTRAN}"
						LINK_FLAGS "-c" 
						LINKER_LANGUAGE FORTRAN)

TARGET_INCLUDE_DIRECTORIES(MUMPS_COMMON_OBJ_FORTRAN	PUBLIC
						"${MUMPS_INCLUDE}"
						"${LIBSEQ_INCLUDE}")
						
# Common MUMPS object files (C)
ADD_LIBRARY(MUMPS_COMMON_OBJ_C OBJECT
						${MUMPS_COMMON_OTHER_HEAD} # not sure IF needed
						${MUMPS_COMMON_OTHER_SRC_C})
				

SET_TARGET_PROPERTIES(MUMPS_COMMON_OBJ_C PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_C} ${ORDERINGS_COMPILER_FLAGS_C}"
						LINK_FLAGS "-c" 
						COMPILE_DEFINITIONS "${COMPILER_DEFS_C}"
						LINKER_LANGUAGE C)

TARGET_INCLUDE_DIRECTORIES(MUMPS_COMMON_OBJ_C	PUBLIC
						"${MUMPS_INCLUDE}"
						"${LIBSEQ_INCLUDE}"
						"${ORDERINGS_INCLUDE}")

# Common MUMPS library
ADD_LIBRARY(libmumps_common STATIC
						$<TARGET_OBJECTS:MUMPS_COMMON_OBJ_FORTRAN>
						$<TARGET_OBJECTS:MUMPS_COMMON_OBJ_C>)

									
TARGET_INCLUDE_DIRECTORIES(libmumps_common	PUBLIC
						"${MUMPS_INCLUDE}"
						"${LIBSEQ_INCLUDE}"
						"${ORDERINGS_INCLUDE}")

TARGET_LINK_LIBRARIES(libmumps_common
						libpord
						libmpiseq
						"${LIBSEQ_LIB}")
		
# Arithmetic-specific MUMPS files
SET(${ARITH_PREFIX}MUMPS_SRC
		src/${ARITH_PREFIX}ana_aux_par.F 
		src/${ARITH_PREFIX}ana_lr.F
		src/${ARITH_PREFIX}fac_asm_master_m.F
		src/${ARITH_PREFIX}fac_asm_master_ELT_m.F
		src/${ARITH_PREFIX}omp_tps_m.F
		src/${ARITH_PREFIX}mumps_comm_buffer.F
		src/${ARITH_PREFIX}mumps_load.F
		src/${ARITH_PREFIX}mumps_lr_data_m.F
		src/${ARITH_PREFIX}mumps_ooc_buffer.F
		src/${ARITH_PREFIX}mumps_ooc.F
		src/${ARITH_PREFIX}mumps_struc_def.F
		src/${ARITH_PREFIX}static_ptr_m.F
		src/${ARITH_PREFIX}lr_core.F
		src/${ARITH_PREFIX}fac_lr.F
		src/${ARITH_PREFIX}fac_omp_m.F
		src/${ARITH_PREFIX}fac_par_m.F
		src/${ARITH_PREFIX}fac_front_LU_type1.F
		src/${ARITH_PREFIX}fac_front_LU_type2.F
		src/${ARITH_PREFIX}fac_front_LDLT_type1.F
		src/${ARITH_PREFIX}fac_front_LDLT_type2.F
		src/${ARITH_PREFIX}fac_front_aux.F
		src/${ARITH_PREFIX}fac_front_type2_aux.F)
			
SET(${ARITH_PREFIX}MUMPS_OTHER_SRC
		src/${ARITH_PREFIX}ini_driver.F
		src/${ARITH_PREFIX}ana_driver.F
		src/${ARITH_PREFIX}fac_driver.F
		src/${ARITH_PREFIX}sol_driver.F
		src/${ARITH_PREFIX}end_driver.F
		src/${ARITH_PREFIX}ana_aux_ELT.F
		src/${ARITH_PREFIX}ana_aux.F
		src/${ARITH_PREFIX}ana_dist_m.F
		src/${ARITH_PREFIX}ana_LDLT_preprocess.F
		src/${ARITH_PREFIX}ana_reordertree.F
		src/${ARITH_PREFIX}arrowheads.F
		src/${ARITH_PREFIX}bcast_int.F
		src/${ARITH_PREFIX}fac_asm_ELT.F
		src/${ARITH_PREFIX}fac_asm.F
		src/${ARITH_PREFIX}fac_b.F
		src/${ARITH_PREFIX}fac_distrib_distentry.F
		src/${ARITH_PREFIX}fac_distrib_ELT.F
		src/${ARITH_PREFIX}fac_lastrtnelind.F
		src/${ARITH_PREFIX}fac_mem_alloc_cb.F
		src/${ARITH_PREFIX}fac_mem_compress_cb.F
		src/${ARITH_PREFIX}fac_mem_free_block_cb.F
		src/${ARITH_PREFIX}fac_mem_stack_aux.F
		src/${ARITH_PREFIX}fac_mem_stack.F
		src/${ARITH_PREFIX}fac_process_band.F
		src/${ARITH_PREFIX}fac_process_blfac_slave.F
		src/${ARITH_PREFIX}fac_process_blocfacto_LDLT.F
		src/${ARITH_PREFIX}fac_process_blocfacto.F
		src/${ARITH_PREFIX}fac_process_bf.F
		src/${ARITH_PREFIX}fac_process_end_facto_slave.F
		src/${ARITH_PREFIX}fac_process_contrib_type1.F
		src/${ARITH_PREFIX}fac_process_contrib_type2.F
		src/${ARITH_PREFIX}fac_process_contrib_type3.F
		src/${ARITH_PREFIX}fac_process_maprow.F
		src/${ARITH_PREFIX}fac_process_master2.F
		src/${ARITH_PREFIX}fac_process_message.F
		src/${ARITH_PREFIX}fac_process_root2slave.F
		src/${ARITH_PREFIX}fac_process_root2son.F
		src/${ARITH_PREFIX}fac_process_rtnelind.F
		src/${ARITH_PREFIX}fac_root_parallel.F
		src/${ARITH_PREFIX}fac_scalings.F
		src/${ARITH_PREFIX}fac_determinant.F
		src/${ARITH_PREFIX}fac_scalings_simScaleAbs.F
		src/${ARITH_PREFIX}fac_scalings_simScale_util.F
		src/${ARITH_PREFIX}fac_sol_pool.F
		src/${ARITH_PREFIX}fac_type3_symmetrize.F
		src/${ARITH_PREFIX}ini_defaults.F
		src/${ARITH_PREFIX}mumps_driver.F
		src/${ARITH_PREFIX}mumps_f77.F
		src/${ARITH_PREFIX}mumps_iXamax.F
		src/${ARITH_PREFIX}ana_mtrans.F
		src/${ARITH_PREFIX}ooc_panel_piv.F
		src/${ARITH_PREFIX}rank_revealing.F
		src/${ARITH_PREFIX}sol_aux.F
		src/${ARITH_PREFIX}sol_bwd_aux.F
		src/${ARITH_PREFIX}sol_bwd.F
		src/${ARITH_PREFIX}sol_c.F
		src/${ARITH_PREFIX}sol_fwd_aux.F
		src/${ARITH_PREFIX}sol_fwd.F
		src/${ARITH_PREFIX}sol_matvec.F
		src/${ARITH_PREFIX}sol_root_parallel.F
		src/${ARITH_PREFIX}tools.F
		src/${ARITH_PREFIX}type3_root.F
		src/${ARITH_PREFIX}mumps_restart.F)
			
# Arithmetic-specific MUMPS object files
ADD_LIBRARY(MUMPS_SPECIFIC_OBJ_FORTRAN OBJECT
						"${MUMPS_COMMON_OTHER_HEAD}" # not sure IF needed
						"${${ARITH_PREFIX}MUMPS_SRC}"
						"${${ARITH_PREFIX}MUMPS_OTHER_SRC}")
							
SET_TARGET_PROPERTIES(MUMPS_SPECIFIC_OBJ_FORTRAN PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_FORTRAN} ${ORDERINGS_COMPILER_FLAGS_FORTRAN}"
						LINK_FLAGS "-c"
						LINKER_LANGUAGE FORTRAN)
			
TARGET_INCLUDE_DIRECTORIES(MUMPS_SPECIFIC_OBJ_FORTRAN PUBLIC
						"${MUMPS_INCLUDE}"
						"${LIBSEQ_INCLUDE}") #aka INCSEQ
							
# Arithmetic-specific MUMPS library
ADD_LIBRARY("lib${ARITH_PREFIX}mumps" STATIC
						$<TARGET_OBJECTS:MUMPS_SPECIFIC_OBJ_FORTRAN>
						$<TARGET_OBJECTS:MUMPS_COMMON_OBJ_C>)
						
TARGET_LINK_LIBRARIES("lib${ARITH_PREFIX}mumps" 
						libmumps_common
						libpord)
		
						
TARGET_INCLUDE_DIRECTORIES("lib${ARITH_PREFIX}mumps" PUBLIC
						"${MUMPS_INCLUDE}"
						"${LIBSEQ_INCLUDE}") #aka INCSEQ
									
			
SET_PROPERTY(TARGET libpord	lib${ARITH_PREFIX}mumps	libmumps_common PROPERTY ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}/lib")
SET_PROPERTY(TARGET libpord	lib${ARITH_PREFIX}mumps	libmumps_common	PROPERTY ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}/lib")
SET_PROPERTY(TARGET libpord	lib${ARITH_PREFIX}mumps	libmumps_common	PROPERTY ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_SOURCE_DIR}/lib")

############### Examples ###############
SET(LIBARITHMUMPS 	"${CMAKE_SOURCE_DIR}/lib/lib${ARITH_PREFIX}mumps.lib"
					"${CMAKE_SOURCE_DIR}/lib/libmumps_common.lib")

ADD_EXECUTABLE(${ARITH_PREFIX}simpletest
						examples/${ARITH_PREFIX}simpletest.F
						)

TARGET_LINK_LIBRARIES(${ARITH_PREFIX}simpletest 	
						"${LIBARITHMUMPS}"
						"${LIBSEQ_LIB}" #aka LIBS
						"${ORDERINGS_LIB}" #aka LORDERINGS
						"${BLAS_LIB}" #aka LIBBLAS
						lib${ARITH_PREFIX}mumps)  
															
TARGET_INCLUDE_DIRECTORIES(${ARITH_PREFIX}simpletest PUBLIC
						"${BLAS_INCLUDE}"
						"${LIBSEQ_INCLUDE}"
						"${MUMPS_INCLUDE}")
						
ADD_DEPENDENCIES(${ARITH_PREFIX}simpletest
						"lib${ARITH_PREFIX}mumps"
						libpord
						libmpiseq
						)
									
SET_TARGET_PROPERTIES(${ARITH_PREFIX}simpletest PROPERTIES
						COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_FORTRAN}"
						LINKER_LANGUAGE FORTRAN)
						
SET_PROPERTY(TARGET	${ARITH_PREFIX}simpletest PROPERTY RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/examples")

############### Target output redirection

IF ( ${ARITH_PREFIX} MATCHES "d")

	ADD_EXECUTABLE(c_example
							examples/c_example.c)
						
	TARGET_INCLUDE_DIRECTORIES(c_example PUBLIC
							"${LIBSEQ_INCLUDE}"
							"${MUMPS_INCLUDE}"
							)
							
						
	TARGET_LINK_LIBRARIES(c_example
							"${LIBSEQ_LIB}" #aka LIBS
							"${ORDERINGS_LIB}" #aka LORDERINGS
							"${BLAS_LIB}" #aka LIBBLAS
							"${LIBARITHMUMPS}"
							)
							
	ADD_DEPENDENCIES(c_example
						"lib${ARITH_PREFIX}mumps"
						libpord
						libmpiseq
						)
						
	SET_TARGET_PROPERTIES(c_example PROPERTIES
							COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_C}"
							LINKER_LANGUAGE C)

	SET_PROPERTY(TARGET c_example PROPERTY RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/examples")
	
ENDIF ( ${ARITH_PREFIX}  MATCHES "d")

############

## Installation routine
#INSTALL(TARGETS libpord
#				lib${ARITH_PREFIX}mumps
#				libmumps_common
#		DESTINATION "${CMAKE_SOURCE_DIR}/lib")
#
#INSTALL(TARGETS libmpiseq
#		DESTINATION "${CMAKE_SOURCE_DIR}/libseq")
#
#INSTALL(TARGETS c_example
#				${ARITH_PREFIX}simpletest
#		DESTINATION "${CMAKE_SOURCE_DIR}/examples")
