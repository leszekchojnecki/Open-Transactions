cmake_minimum_required (VERSION 2.6)
project (othint)

# Options

option(LOCAL_EDITLINE "Use local Editline library ($HOME/.local)" OFF)
option(WITH_WRAPPER "Build with wrapper" OFF)
option(LOCAL_OT "Build with locally installed ot" ON)

# Set variables

set(CMAKE_PREFIX_PATH "$ENV{HOME}/.local")
set(CMAKE_SKIP_BUILD_RPATH  FALSE) # Always include RPATH to the binary

if (WITH_WRAPPER)
	set(CMAKE_CXX_COMPILER "$ENV{HOME}/.local/bin/compile-cxx")
	set(CMAKE_CC_COMPILER "$ENV{HOME}/.local/bin/compile-cc")
endif (WITH_WRAPPER)

set(CMAKE_CXX_FLAGS "-std=c++11 -g3 -O0 ")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/") # Add folder with cmake modules

include_directories("${CMAKE_PREFIX_PATH}/include/ot/otlib") # For ExportWrapper.h needed by OTAPI.h

# Add executable

file(GLOB othint_SRC # All files in directory:
	"*.h"
	"*.hpp"
	"*.cpp"
)

add_executable(othint ${othint_SRC})

# Find libraries and headers for Editline

if (LOCAL_EDITLINE)
	find_package(EditlineLocal)
	if (EDITLINE_FOUND)
		include_directories(${EDITLINE_INCLUDE_DIR})
		target_link_libraries (othint ${EDITLINE_LIBRARY})
	endif (EDITLINE_FOUND)
else (LOCAL_EDITLINE)
	find_package(Editline)
	if (EDITLINE_FOUND)
		include_directories(${EDITLINE_INCLUDE_DIR})
		target_link_libraries (othint ${EDITLINE_LIBRARY})
	endif (EDITLINE_FOUND)
endif(LOCAL_EDITLINE)

# Find libraries and headers for OT

if (LOCAL_OT)
	find_package(OTLocal)
	if (OT_FOUND)
		include_directories(${OT_INCLUDE_DIR})
		target_link_libraries (othint ${OT_LIBRARY})
	endif (OT_FOUND)
endif(LOCAL_OT)

# make run

add_custom_target(run
    COMMAND echo "" && ./othint --complete-shell
    DEPENDS othint
    WORKING_DIRECTORY ${CMAKE_PROJECT_DIR}
)
