cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(openfpm_pdata LANGUAGES C CXX)

if (POLICY CMP0074)
        cmake_policy(SET CMP0074 OLD)
endif ()

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake_modules/)

if (POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

set(BOOST_INCLUDE ${Boost_INCLUDE_DIR} CACHE PATH "Include directory for BOOST")
set(HDF5_ROOT CACHE PATH "HDF5 root path")
set(SE_CLASS1 CACHE BOOL "Activate compilation with SE_CLASS1")
set(SE_CLASS3 CACHE BOOL "Activate compilation with SE_CLASS3")
set(ENV{HDF5_ROOT} ${HDF5_ROOT})
set(OPENFPM_PDATA_DIR CACHE PATH "OpenFPM_pdata dirs")
set(PETSC_ROOT CACHE PATH "If compiling with linear algebra indicate the PETSC root directory")
set(ENV{PETSC_DIR} ${PETSC_ROOT})


set (CMAKE_CXX_STANDARD 11)
set (CMAKE_CUDA_STANDARD 11)

if(NOT TINYOBJLOADER_FOUND)
	set(TINYOBJLOADER_ROOT CACHE PATH "TinyObjLoader library path")
endif()

set(ENV{PATH} "$ENV{PATH}:${HDF5_ROOT}/bin")
set(HDF5_PREFER_PARALLEL TRUE)

if(ENABLE_GPU)
	enable_language(CUDA)
	find_package(CUDA)

        if (CUDA_VERSION_MAJOR EQUAL 9 AND CUDA_VERSION_MINOR EQUAL 2)
                message("CUDA is compatible")
                set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=611 --diag_suppress=2885 --diag_suppress=2886  --diag_suppress=2887  --diag_suppress=2888 --diag_suppress=186 --diag_suppress=111 " --expt-extended-lambda)
        elseif ( CUDA_VERSION_MAJOR EQUAL 10 AND CUDA_VERSION_MINOR EQUAL 1 )
                message("CUDA is compatible")
                set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=2915 --diag_suppress=2912 --diag_suppress=2913 --diag_suppress=111 --diag_suppress=186 --diag_suppress=611 " --expt-extended-lambda)
        elseif ( CUDA_VERSION_MAJOR EQUAL 10 AND CUDA_VERSION_MINOR EQUAL 2 )
                message("CUDA is compatible")
                set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=2977 --diag_suppress=2976 --diag_suppress=2979 --diag_suppress=186" --expt-extended-lambda)
        elseif ( CUDA_VERSION_MAJOR EQUAL 11 AND CUDA_VERSION_MINOR EQUAL 0 )
                message("CUDA is compatible 11.0")
                set(WARNING_SUPPRESSION_AND_OPTION_NVCC  -Xcudafe "--display_error_number --diag_suppress=3059 --diag_suppress=3058 --diag_suppress=3057 --diag_suppress=3056 --diag_suppress=611  --diag_suppress=186" --expt-extended-lambda)
	else()
                message(FATAL_ERROR "CUDA is incompatible, version 9.2 10.1 and 10.2 is only supported")
        endif()
endif()

find_package(Boost 1.66.0 REQUIRED COMPONENTS unit_test_framework iostreams program_options)
find_package(MPI REQUIRED)
find_package(HDF5 REQUIRED)
find_package(TinyObjLoader)
find_package(PETSc)

if(CUDA_FOUND)
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cuda.cu")
else()
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cuda.cpp")
endif()

###### CONFIG.h FILE ######

if(SE_CLASS1)
	set(DEFINE_SE_CLASS1 "#define SE_CLASS1")
endif()

if(SE_CLASS3)
	set(DEFINE_SE_CLASS3 "#define SE_CLASS3")
endif()

if(CUDA_FOUND)
	set(DEFINE_CUDA_GPU "#define CUDA_GPU")
endif()

if(MPI_FOUND)
	set(DEFINE_HAVE_MPI "#define HAVE_MPI")
else()
	message( FATAL_ERROR "MPI is required in order to install OpenFPM" )
endif()

if(PETSC_FOUND)
        set(DEFINE_HAVE_PETSC "#define HAVE_PETSC")
endif()

if (Boost_FOUND)
	set(DEFINE_HAVE_BOOST "#define HAVE_BOOST")
	set(DEFINE_HAVE_BOOST_IOSTREAMS "#define HAVE_BOOST_IOSTREAMS")
	set(DEFINE_HAVE_BOOST_PROGRAM_OPTIONS "#define HAVE_BOOST_PROGRAM_OPTIONS")
	set(DEFINE_HAVE_BOOST_UNIT_TEST_FRAMEWORK "#define HAVE_BOOST_UNIT_TEST_FRAMEWORK")
else()
	message( FATAL_ERROR "BOOST is required in order to install OpenFPM" )
endif()

if(HDF5_FOUND)
	if (HDF5_IS_PARALLEL)
		set(DEFINE_HAVE_HDF5 "#define HAVE_HDF5")
	else()
		message( STATUS "HDF5 found ${HDF5_INCLUDE_DIRS} does not have parallel support, OpenFPM require it" )
	endif()
else()
	message( FATAL_ERROR "HDF5 with parallel support is required in order to install OpenFPM" )
endif()

if(TINYOBJLOADER_FOUND)
        set(DEFINE_HAVE_TINYOBJLOADER "#define HAVE_TINYOBJLOADER 1")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/config/config_cmake.h.in ${CMAKE_CURRENT_SOURCE_DIR}/src/config/config.h)

include_directories(SYSTEM ${MPI_INCLUDE_PATH})

add_subdirectory (src)

