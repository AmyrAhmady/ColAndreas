cmake_minimum_required (VERSION 2.8.11)

project(ColAndreas)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CA_VERSION "v1.5.0")
set(CA_DATABASE_VERSION 2)
add_definitions(-DCA_VERSION="${CA_VERSION}" -DCA_DATABASE_VERSION=${CA_DATABASE_VERSION})

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING " " FORCE)
endif(NOT CMAKE_BUILD_TYPE)

option(BuildWizard "Compile the Wizard application." OFF)

if(WIN32)
	add_definitions(-DWIN32 -D_WIN32)
else()
	add_definitions(-DLINUX)

	set(CMAKE_C_FLAGS "-m32")
	set(CMAKE_CXX_FLAGS "-m32 -std=c++11 -Wno-write-strings")
	set(CMAKE_SHARED_LINKER_FLAGS "-m32 -fshort-wchar -shared")
endif(WIN32)

# Bullet defs
set(BUILD_BULLET2_DEMOS OFF CACHE BOOL "BUILD_BULLET2_DEMOS" FORCE)
set(BUILD_EXTRAS OFF CACHE BOOL "BUILD_EXTRAS" FORCE)
set(BUILD_BULLET3 OFF CACHE BOOL "BUILD_BULLET3" FORCE)
set(BUILD_UNIT_TESTS OFF CACHE BOOL "BUILD_UNIT_TESTS" FORCE)

add_subdirectory(deps/bullet3)

if(WIN32)
	#  Taken from bullet engine cmake, they do it this way so we must follow it to avoid conflicts with RunTimeLibrary issues
	if(NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
		#We statically link to reduce dependancies
		foreach(flag_var CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO )
			if(${flag_var} MATCHES "/MD")
				string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
			endif(${flag_var} MATCHES "/MD")
			if(${flag_var} MATCHES "/MDd")
				string(REGEX REPLACE "/MDd" "/MTd" ${flag_var} "${${flag_var}}")
			endif(${flag_var} MATCHES "/MDd")
		endforeach(flag_var)
	endif(NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
endif()

include_directories(
	"${CMAKE_CURRENT_SOURCE_DIR}/src"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/SDK"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/SDK/amx"
	"${CMAKE_CURRENT_SOURCE_DIR}/deps/bullet3/src"
)

file(GLOB CA_Source
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/SDK/*.cpp"
)

if(WIN32)
	set(DEF_FILE ${CMAKE_CURRENT_SOURCE_DIR}/src/ColAndreas.def)
else()
	set(DEF_FILE "")
endif()

add_library(ColAndreas SHARED ${CA_Source} ${DEF_FILE})
set_target_properties(ColAndreas PROPERTIES PREFIX "")

target_link_libraries(ColAndreas BulletCollision BulletDynamics BulletSoftBody LinearMath)

if(BuildWizard)
	add_subdirectory(WizardApp)
endif()
