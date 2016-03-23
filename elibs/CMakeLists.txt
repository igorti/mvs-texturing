cmake_minimum_required(VERSION 3.0)
project(elibs)
include(ExternalProject)

if(RESEARCH)
    ExternalProject_Add(ext_gco
        PREFIX          gco
        URL             http://vision.csd.uwo.ca/code/gco-v3.0.zip
        URL_MD5         10e071892c38f076d30a91ca5351a847
        UPDATE_COMMAND  ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/elibs/CMakeLists/gco.txt CMakeLists.txt
        SOURCE_DIR      ${CMAKE_SOURCE_DIR}/elibs/gco
        INSTALL_COMMAND ""
    )
endif()

#find_package eigen
ExternalProject_Add(ext_eigen
    PREFIX          eigen
    URL             https://bitbucket.org/eigen/eigen/get/3.2.5.tar.bz2
    URL_MD5         21a928f6e0f1c7f24b6f63ff823593f5
    SOURCE_DIR      ${CMAKE_SOURCE_DIR}/elibs/eigen
    CONFIGURE_COMMAND ""
    BUILD_COMMAND   ""
    INSTALL_COMMAND ""
)

ExternalProject_Add(ext_mve
    PREFIX          mve
    GIT_REPOSITORY  https://github.com/andre-schulz/mve.git
    GIT_TAG         cmake
    UPDATE_COMMAND  ""
    SOURCE_DIR      ${CMAKE_SOURCE_DIR}/elibs/mve
    CONFIGURE_COMMAND ""
    BUILD_COMMAND   ""
    #BUILD_IN_SOURCE 1
    INSTALL_COMMAND ""
)


set(LIBJPEG_FILENAME "jpegsr9b.zip")
set(LIBPNG_FILENAME "lpng1621.zip")
set(ZLIB_FILENAME "zlib128.zip")
set(LIBTIFF_FILENAME "tiff-4.0.4.zip")

set(LIBJPEG_URL "http://www.ijg.org/files/${LIBJPEG_FILENAME}")
set(LIBPNG_URL "https://download.sourceforge.net/libpng/${LIBPNG_FILENAME}")
set(ZLIB_URL "https://download.sourceforge.net/libpng/${ZLIB_FILENAME}")
set(LIBTIFF_URL "http://download.osgeo.org/libtiff/${LIBTIFF_FILENAME}")
set(PATCHES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/patches")

file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/bin")
file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include")
file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/lib")

message("Forcing build type to \"Release\"")
message("Reminder: Select Release configuration in Visual Studio before building")
set(CMAKE_BUILD_TYPE Release CACHE STRING "")

find_file(WIN32_MAK_LOCATION win32.mak
	DOC "Location of win32.mak"
	PATHS
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.0A\\Include"
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.1\\Include"
		"C:\\Program Files (x86)\\Microsoft SDKs\\Windows\\v7.1A\\Include"
		ENV Path)
ExternalProject_Add(libjpeg
	PREFIX build/libjpeg
	URL ${LIBJPEG_URL}
	URL_HASH SHA256=d98f2a927bd17a58fe14781ec7957b5d514a8e5f04c0d071e65b967bf34e19d4
	CONFIGURE_COMMAND nmake /f makefile.vc setup-v10
	BUILD_COMMAND nmake /f makefile.vc
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jconfig.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jerror.h   ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jmorecfg.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jpegint.h  ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/jpeglib.h  ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libjpeg/src/libjpeg/libjpeg.lib  ${CMAKE_CURRENT_SOURCE_DIR}/lib
	BUILD_IN_SOURCE 1)
ExternalProject_Add_Step(libjpeg copy_win32_mak
	COMMAND ${CMAKE_COMMAND} -E copy ${WIN32_MAK_LOCATION} <SOURCE_DIR>
	DEPENDEES download
	DEPENDERS configure)

ExternalProject_Add(zlib
	PREFIX build/zlib
	URL ${ZLIB_URL}
	URL_HASH SHA256=879d73d8cd4d155f31c1f04838ecd567d34bebda780156f0e82a20721b3973d5
	CMAKE_ARGS
		-DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
		-DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_CURRENT_SOURCE_DIR}
	BUILD_IN_SOURCE 1)

ExternalProject_Add(libpng
	DEPENDS zlib
	PREFIX build/libpng
	URL ${LIBPNG_URL}
	URL_HASH SHA256=1e4055c290992e0edf5045b1f78585a026d96f71ad706d6bda290cd9a181f4af
	CMAKE_ARGS
		-DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
		-DCMAKE_INSTALL_PREFIX:STRING=${CMAKE_CURRENT_SOURCE_DIR}
		-DZLIB_ROOT:STRING=${CMAKE_CURRENT_SOURCE_DIR}
	BUILD_IN_SOURCE 1)

ExternalProject_Add(libtiff
	PREFIX build/libtiff
	URL ${LIBTIFF_URL}
	URL_HASH SHA256=4fec3abf1c79b000805446235fecc6b5a764527c5fe30e1f707b251a8720fce3
	CONFIGURE_COMMAND ""
	BUILD_COMMAND nmake /f Makefile.vc lib
	INSTALL_COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffconf.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiff.h     ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffio.h   ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffio.hxx ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/tiffvers.h ${CMAKE_CURRENT_SOURCE_DIR}/include
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff.lib   ${CMAKE_CURRENT_SOURCE_DIR}/lib
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff_i.lib ${CMAKE_CURRENT_SOURCE_DIR}/lib
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/build/libtiff/src/libtiff/libtiff/libtiff.dll ${CMAKE_CURRENT_SOURCE_DIR}/bin
	BUILD_IN_SOURCE 1)
