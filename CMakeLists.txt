cmake_minimum_required(VERSION 3.0)

set(SHINE_VERSION_MAJOR 3)
set(SHINE_VERSION_MINOR 1)
set(SHINE_VERSION_MICRO 1)
set(SHINE_VERSION ${SHINE_VERSION_MAJOR}.${SHINE_VERSION_MINOR}.${SHINE_VERSION_MICRO})
set(VERSION ${SHINE_VERSION})

project(shine VERSION ${SHINE_VERSION})

option(BUILD_SHARED_LIBS "Build shared library." ON)
option(BUILD_STATIC_LIBS "Build static library." ON)

if (${CMAKE_VERSION} VERSION_LESS "3.20")
    include (TestBigEndian)
    TEST_BIG_ENDIAN(IS_BIG_ENDIAN)
    if(IS_BIG_ENDIAN)
        add_definitions(-DSHINE_BIG_ENDIAN)
    endif()
else()
    if (${CMAKE_C_BYTE_ORDER} STREQUAL BIG_ENDIAN)
        add_definitions(-DSHINE_BIG_ENDIAN)
    endif()
endif()

include(CheckIncludeFiles)

CHECK_INCLUDE_FILES(byteswap.h HAVE_BYTESWAP_H)
if (HAVE_BYTESWAP_H)
    add_definitions(SHINE_HAVE_BSWAP_H)
endif()

set(LIB_OBJS src/lib/bitstream.c src/lib/huffman.c
src/lib/l3bitstream.c src/lib/l3loop.c src/lib/l3mdct.c
src/lib/l3subband.c src/lib/layer3.c src/lib/reservoir.c
src/lib/tables.c)

set(PUBLIC_HEADERS src/lib/layer3.h)

if (MSVC AND BUILD_SHARED_LIBS)
    configure_file(shine.rc.in shine.rc)
    set(RC_FILE ${CMAKE_CURRENT_BINARY_DIR}/shine.rc)
endif()

if (BUILD_SHARED_LIBS)
    add_library(libshine SHARED ${LIB_OBJS} ${RC_FILE})
    set_target_properties(libshine PROPERTIES OUTPUT_NAME shine)
    if (MSVC)
        target_link_options(libshine PRIVATE /DEF:${CMAKE_CURRENT_SOURCE_DIR}/libshine.def)
    endif()
    list(APPEND TARGETS libshine)
endif()
if (BUILD_STATIC_LIBS)
    add_library(libshine-static STATIC ${LIB_OBJS})
    if (NOT MSVC OR (MSVC AND NOT BUILD_SHARED_LIBS))
        set_target_properties(libshine-static PROPERTIES OUTPUT_NAME shine)
    else()
        set_target_properties(libshine-static PROPERTIES OUTPUT_NAME shine-static)
    endif()
    list(APPEND TARGETS libshine-static)
endif()

set(prefix ${CMAKE_PREFIX_PATH})
set(CMAKE_INSTALL_INCLUDEDIR include/shine)

if (MSVC)
    configure_file(shine_msvc.pc.in shine.pc @ONLY)
else()
    configure_file(shine.pc.in shine.pc @ONLY)
endif()

install(TARGETS ${TARGETS})
install(FILES ${PUBLIC_HEADERS} TYPE INCLUDE)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/shine.pc DESTINATION lib/pkgconfig)
