CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(MumpsProject C)

# Arithmetic configuration
SET(ARITHMETIC "double" CACHE STRING "MUMPS arithmetic")
SET_PROPERTY(CACHE ARITHMETIC PROPERTY STRINGS float double complex complex-double)


IF (ARITHMETIC MATCHES "float")
	SET(ARITH_PREFIX "s")
	MESSAGE("No float library implemented yet")
ELSEIF (ARITHMETIC MATCHES "double")
	SET(ARITH_PREFIX "d")
ELSEIF (ARITHMETIC MATCHES "complex")
	SET(ARITH_PREFIX "c")
	MESSAGE("No complex-float library implemented yet")
ELSEIF (ARITHMETIC MATCHES "complex-double")
	SET(ARITH_PREFIX "z")
	MESSAGE("No complex-double library implemented yet")
ENDIF ()

MESSAGE(STATUS "Arithmetic set to '${ARITHMETIC}'. Added '${ARITH_PREFIX}' prefix...")


##########################

SET(MUMPS_ROOT_DIRECTORY "D:/SourceTreeWS/MumpsWin/Mumps5.0.0" CACHE PATH "Mumps root directory" FORCE)
SET(USE_MPI FALSE CACHE BOOL "Select if MPI is needed" FORCE)
SET(USE_DEFAULT_ORDERING TRUE CACHE BOOL "Select if default ordering library should be used" FORCE)
SET(USE_MKL_BLAS TRUE CACHE BOOL "Select if default MKL BLAS should be used" FORCE)

SET(MUMPS_INCLUDE
	"${MUMPS_ROOT_DIRECTORY}/include")
	
SET(MUMPS_LIBRARIES
	"${MUMPS_ROOT_DIRECTORY}/lib/lib${ARITH_PREFIX}mumps.lib"
	"${MUMPS_ROOT_DIRECTORY}/lib/libmumps_common.lib")


IF (USE_MPI)
	MESSAGE("MPI is not implemented yet.")
ELSE (USE_MPI)
	SET(LIBSEQ_INCLUDE "${MUMPS_ROOT_DIRECTORY}/libseq")
	SET(LIBSEQ_LIBRARIES "${MUMPS_ROOT_DIRECTORY}/libseq/libmpiseq.lib")
ENDIF (USE_MPI)

IF (USE_DEFAULT_ORDERING)
	SET(ORDERINGS_LIBRARIES	"${MUMPS_ROOT_DIRECTORY}/lib/libpord.lib")
ELSE (USE_DEFAULT_ORDERING)
	SET(ORDERINGS_LIBRARIES "" CACHE PATH "Path to ordering library" FORCE)
ENDIF (USE_DEFAULT_ORDERING)

IF (USE_MKL_BLAS)
	SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}")
	
	SET(BLAS_LIBRARIES
		"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_intel_lp64.lib"
		"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_intel_thread.lib"
		"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/lib/intel64/mkl_core.lib"
		"C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/intel64/libiomp5md.lib"
		)
		
	SET(BLAS_INCLUDE
		"C:/Program Files (x86)/Intel/Composer XE 2015/mkl/include"
		"C:/Program Files (x86)/Intel/Composer XE 2015/compiler/include"
		)
		
ELSE (USE_MKL_BLAS)
	SET(BLAS_ROOT "" CACHE PATH "Path to BLAS directory" FORCE)
	SET(BLAS_LIBRARIES "${BLAS_ROOT}/lib/lib.lib")
	SET(BLAS_INCLUDE "${BLAS_ROOT}/include")
ENDIF (USE_MKL_BLAS)

################ Project configuration
LINK_DIRECTORIES("C:/Program Files (x86)/Intel/Composer XE 2015/compiler/lib/intel64")
ADD_EXECUTABLE(mumps_project1
						"src/mumps_project1")
					
TARGET_INCLUDE_DIRECTORIES(mumps_project1 PUBLIC
							"${LIBSEQ_INCLUDE}"
							"${MUMPS_INCLUDE}")
					
TARGET_LINK_LIBRARIES(mumps_project1 	
						"${MUMPS_LIBRARIES}"
						"${LIBSEQ_LIBRARIES}" #aka LIBS
						"${ORDERINGS_LIBRARIES}" #aka LORDERINGS
						"${BLAS_LIBRARIES}" #aka LIBBLAS
						)
						

						
SET(GENERIC_COMPILER_FLAGS_C #aka OPTC
			"-MD")						
					
SET_TARGET_PROPERTIES(mumps_project1 PROPERTIES
						# COMPILE_FLAGS "${GENERIC_COMPILER_FLAGS_C}"
						LINKER_LANGUAGE C
						)