cmake_minimum_required(VERSION 3.1)

project(Texturing)
include(ExternalProject)

if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'RELWITHDEBINFO' as none was specified.")
    set(CMAKE_BUILD_TYPE RELWITHDEBINFO)
endif()


set(CMAKE_CXX_STANDARD 11)

if(WIN32 AND NOT UNIX)
	if(MSVC)
		set(MVE_BUILD_WITH_SSE2 yes CACHE BOOL "Build MVE with SSE2 support")
		set(MVE_BUILD_WITH_SSE3 yes CACHE BOOL "Build MVE with SSE3 support")
		if(MVE_BUILD_WITH_SSE2)
			add_definitions(-D__SSE2__)
		endif()
		if(MVE_BUILD_WITH_SSE3)
			add_definitions(-D__SSE3__)
		endif()
	endif()

	set(CMAKE_PREFIX_PATH "${CMAKE_CURRENT_SOURCE_DIR}/3rdparty" "${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/qt5/src/qt5/qtbase/lib/cmake/Qt5")
	set(JPEG_NAMES "libjpeg")
    set(CPACK_GENERATOR "ZIP")
else(WIN32 AND NOT UNIX)
    set(CPACK_GENERATOR "TGZ")
endif(WIN32 AND NOT UNIX)
include(CPack)

set(RESEARCH "OFF" CACHE BOOL "Use the gco library for Multi-Label Optimization, which is licensed only for research purposes!")
if(RESEARCH)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DRESEARCH=1")
endif()

FIND_PACKAGE(OpenMP)
if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wundef -pedantic")
endif()

find_package(PNG REQUIRED)
find_package(JPEG REQUIRED)
find_package(TIFF REQUIRED)

add_definitions(${PNG_DEFINITIONS})
if(WIN32)
    add_definitions(-DGLEW_STATIC)
	add_definitions(-DWIN32_LEAN_AND_MEAN -DVC_EXTRALEAN -DNOMINMAX)
endif()

add_subdirectory(elibs)
include_directories(SYSTEM
    ${CMAKE_SOURCE_DIR}/elibs/mve/libs
    ${CMAKE_SOURCE_DIR}/elibs/eigen
)

include_directories(${JPEG_INCLUDE_DIR})
include_directories(${PNG_INCLUDE_DIRS})
include_directories(${TIFF_INCLUDE_DIR})
include_directories(libs)

link_directories(
    ${CMAKE_SOURCE_DIR}/elibs/mve/libs/mve
    ${CMAKE_SOURCE_DIR}/elibs/mve/libs/util
)

if(RESEARCH)
    include_directories(SYSTEM ${CMAKE_SOURCE_DIR}/elibs/gco)
    link_directories(${CMAKE_BINARY_DIR}/elibs/gco/src/ext_gco-build)
endif()

if(WIN32)
	set(LIBPNG_DLL "elibs/bin/libpng16.dll")
	set(ZLIB_DLL "elibs/bin/zlib.dll")
	set(LIBTIFF_DLL "elibs/bin/libtiff.dll")
	install(FILES ${LIBPNG_DLL} ${ZLIB_DLL} ${LIBTIFF_DLL} DESTINATION .)
endif()

add_subdirectory(libs)
add_subdirectory(apps)

if(RESEARCH)
    message(
"
******************************************************************************
 Due to use of the -DRESEARCH=ON option, the resulting program is licensed
 for research purposes only. Please pay special attention to the gco license.
******************************************************************************
")
endif()
