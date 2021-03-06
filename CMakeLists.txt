cmake_minimum_required( VERSION 3.0.1 )
project( usbcan )

set( EXECUTABLE_OUTPUT_PATH bin )
set( LIBRARY_OUTPUT_PATH lib )

set( CMAKE_BUILD_TYPE Release )
set( CMAKE_SKIP_BUILD_RPATH true )
set( CMAKE_C_STANDARD 99 )
set( CMAKE_C_FLAGS "-Wall -Wextra -pedantic")

set( PROJECT_LINK_LIBS ${PROJECT_LINK_LIBS} Ginkgo_Driver pthread )

# Architecture detection courtesy of BoringSSL
if (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
    set(ARCH "x86_64")
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "amd64")
    set(ARCH "x86_64")
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "AMD64")
    # cmake reports AMD64 on Windows, but we might be building for 32-bit.
    if (CMAKE_CL_64)
	set(ARCH "x86_64")
    else()
	set(ARCH "x86")
    endif()
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86")
    set(ARCH "x86")
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "i386")
    set(ARCH "x86")
elseif (${CMAKE_SYSTEM_PROCESSOR} STREQUAL "i686")
    set(ARCH "x86")
elseif (${CMAKE_SYSTEM_PROCESSOR} MATCHES "^arm")
    set(ARCH "arm")
else()
    message(FATAL_ERROR "Unknown processor:" ${CMAKE_SYSTEM_PROCESSOR})
endif()

if( ${CMAKE_SYSTEM_NAME} MATCHES "Darwin" )
    set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DOS_UNIX" )
    link_directories( lib/macos ./ )
elseif( ${CMAKE_SYSTEM_NAME} MATCHES "Linux" )
    set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DOS_UNIX -D_POSIX_C_SOURCES=200809L" )
    set( PROJECT_LINK_LIBS ${PROJECT_LINK_LIBS} usb )
    if( ${ARCH} STREQUAL "arm" )
	link_directories( /lib/arm-linux-gnueabihf lib/rpi ./ )
    elseif( ${ARCH} STREQUAL "x86_64" )
        link_directories( lib/linux_64bit ./ )
    else()
	link_directories( lib/linux_32bit ./ )
    endif(${ARCH} STREQUAL "arm" )
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

message(STATUS "Building for ${CMAKE_SYSTEM_NAME}/${ARCH}")

include_directories(include)
set( LIB_SOURCES src/usbcan.c )

add_library( usbcan SHARED ${LIB_SOURCES} )
target_link_libraries( usbcan ${PROJECT_LINK_LIBS} )

set( UTIL_LINK_LIBS ${PROJECT_LINK_LIBS} usbcan )
set( DUMP_SOURCES utils/getopt.c utils/usbcandump.c )
add_executable( usbcandump ${DUMP_SOURCES} )
target_link_libraries( usbcandump ${UTIL_LINK_LIBS} )

set( FLOOD_SOURCES utils/getopt.c utils/usbcanflood.c )
add_executable( usbcanflood ${FLOOD_SOURCES} )
target_link_libraries( usbcanflood ${UTIL_LINK_LIBS} )

install(TARGETS usbcandump usbcanflood usbcan
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib)
