cmake_minimum_required(VERSION 3.4.3)
if (DEFINED LLVM_DIR)
        set(ENV{LLVM_DIR} "${LLVM_DIR}")
    endif()
    if (DEFINED ENV{LLVM_DIR})
        # We need to match the build environment for LLVM:
        # In particular, we need C++11 and the -fno-rtti flag
        set(CMAKE_CXX_STANDARD 14)
	if(CMAKE_BUILD_TYPE MATCHES "Debug")
		set(CMAKE_CXX_FLAGS "-fPIC -std=gnu++14 -O0 -fno-rtti -Wno-deprecated")
	else()
		set(CMAKE_CXX_FLAGS "-fPIC -std=gnu++14 -O3 -fno-rtti -Wno-deprecated")
	endif()
	set(CMAKE_C_FLAGS "-fPIC")

find_package(LLVM REQUIRED CONFIG)
    
    list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
    include(AddLLVM)

    add_definitions(${LLVM_DEFINITIONS})
    include_directories(${LLVM_INCLUDE_DIRS})

else()
    message(FATAL_ERROR "\
WARNING: The LLVM_DIR var was not set (required for an out-of-source build)!\n\
Please set this to environment variable to point to the LLVM build directory\
(e.g. on linux: export LLVM_DIR=/path/to/llvm/build/dir)")
endif()

if (EXISTS "${SVF_DIR}")
else()
    set(SVF_DIR $ENV{SVF_DIR})
    if(EXISTS "${SVF_DIR}")
    else()
    message(FATAL_ERROR "\
WARNING: The SVF_DIR var was not set (required for an out-of-source build)!\n\
Please set this to environment variable to point to the SVF_DIR directory or set this variable to cmake configuration\n
(e.g. on linux: export SVF_DIR=/path/to/SVF/dir) \n or \n \n(make the project via: cmake -DSVF_DIR=your_path_to_SVF) ")
    endif()
endif()

if(CMAKE_BUILD_TYPE MATCHES "Debug")
    MESSAGE (STATUS "building SVF in debug mode")
    if (EXISTS "${SVF_DIR}/Debug-build")
        set(SVF_LIB "${SVF_DIR}/Debug-build/lib/libSvf.a")
    	set(LLVMCudd "${SVF_DIR}/Debug-build/lib/CUDD/libCudd.a")
    else()
        set(SVF_LIB "${SVF_DIR}/Release-build/lib/libSvf.a")
        set(LLVMCudd "${SVF_DIR}/Release-build/lib/CUDD/libCudd.a")
    endif()
else()
    MESSAGE (STATUS "building SVF in release mode")
    set(SVF_LIB "${SVF_DIR}/Release-build/lib/libSvf.a")
    set(LLVMCudd "${SVF_DIR}/Release-build/lib/CUDD/libCudd.a")
endif()
set(SVF_HEADER "${SVF_DIR}/include")
include_directories(${SVF_HEADER})

# set z3 env
if (DEFINED Z3_DIR)
    set(ENV{Z3_DIR} "${Z3_DIR}")
else()
    if (DEFINED ENV{Z3_DIR})
        set(Z3_DIR $ENV{Z3_DIR})
    else()
    message(FATAL_ERROR "\
WARNING: The Z3_DIR var was not set (required for an out-of-source build)!\n\
Please set this to environment variable to point to the SVF_DIR directory or set this variable to cmake configuration\n
(e.g. on linux: export Z3_DIR=/path/to/Z3/dir) \n or \n \n(make the project via: cmake -DZ3_DIR=your_path_to_Z3) ")
    endif()
endif()
set(Z3_HEADER "${Z3_DIR}/include/")
include_directories(${Z3_HEADER})
LINK_DIRECTORIES(${Z3_DIR}/bin)

# add_subdirectory(src)

llvm_map_components_to_libnames(llvm_libs bitwriter core ipo irreader instcombine instrumentation target linker analysis scalaropts support )
file (GLOB SOURCES
	svf-collect-info.cpp
)
add_executable(svf-collect ${SOURCES})

target_link_libraries(svf-collect ${SVF_LIB} ${LLVMCudd} ${llvm_libs})
target_link_libraries(svf-collect ${Z3_DIR}/bin/libz3.a )
set_target_properties( svf-collect PROPERTIES
                       RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin )