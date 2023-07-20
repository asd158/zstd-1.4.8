CMAKE_MINIMUM_REQUIRED(VERSION 3.19)
# ################################################################
# Copyright (c) 2015-present, Yann Collet, Facebook, Inc.
# All rights reserved.
#
# This source code is licensed under both the BSD-style license (found in the
# LICENSE file in the root directory of this source tree) and the GPLv2 (found
# in the COPYING file in the root directory of this source tree).
# ################################################################
PROJECT(libzstd)
SET(CMAKE_INCLUDE_CURRENT_DIR TRUE)
OPTION(ZSTD_BUILD_STATIC "BUILD STATIC LIBRARIES" ON)
SET(LIBRARY_DIR lib)
# Define library directory, where sources and header files are located
INCLUDE_DIRECTORIES(${LIBRARY_DIR} ${LIBRARY_DIR}/common)
FILE(GLOB CommonSources ${LIBRARY_DIR}/common/*.c)
FILE(GLOB CompressSources ${LIBRARY_DIR}/compress/*.c)
FILE(GLOB DecompressSources ${LIBRARY_DIR}/decompress/*.c)
FILE(GLOB DictBuilderSources ${LIBRARY_DIR}/dictBuilder/*.c)
FILE(GLOB DeprecatedSources ${LIBRARY_DIR}/deprecated/*.c)
SET(Sources
    ${CommonSources}
    ${CompressSources}
    ${DecompressSources}
    ${DictBuilderSources}
    ${DeprecatedSources})
FILE(GLOB CommonHeaders ${LIBRARY_DIR}/common/*.h)
FILE(GLOB CompressHeaders ${LIBRARY_DIR}/compress/*.h)
FILE(GLOB DecompressHeaders ${LIBRARY_DIR}/decompress/*.h)
FILE(GLOB DictBuilderHeaders ${LIBRARY_DIR}/dictBuilder/*.h)
FILE(GLOB DeprecatedHeaders ${LIBRARY_DIR}/deprecated/*.h)
SET(Headers
    ${LIBRARY_DIR}/zstd.h
    ${CommonHeaders}
    ${CompressHeaders}
    ${DecompressHeaders}
    ${DictBuilderHeaders}
    ${DeprecatedHeaders})
# Split project to static and shared libraries build
IF (ZSTD_BUILD_STATIC)
    ADD_LIBRARY(libzstd STATIC ${Sources} ${Headers})
    IF (ZSTD_MULTITHREAD_SUPPORT)
        SET_PROPERTY(TARGET libzstd APPEND PROPERTY COMPILE_DEFINITIONS "ZSTD_MULTITHREAD")
        IF (UNIX)
            TARGET_LINK_LIBRARIES(libzstd ${THREADS_LIBS})
        ENDIF ()
    ENDIF ()
ENDIF ()
# Add specific compile definitions for MSVC project
IF (MSVC)
    IF (ZSTD_BUILD_SHARED)
        SET_PROPERTY(TARGET libzstd APPEND PROPERTY COMPILE_DEFINITIONS "ZSTD_DLL_EXPORT=1;ZSTD_HEAPMODE=0;_CONSOLE;_CRT_SECURE_NO_WARNINGS")
    ENDIF ()
    IF (ZSTD_BUILD_STATIC)
        SET_PROPERTY(TARGET libzstd APPEND PROPERTY COMPILE_DEFINITIONS "ZSTD_HEAPMODE=0;_CRT_SECURE_NO_WARNINGS")
    ENDIF ()
ENDIF ()
TARGET_INCLUDE_DIRECTORIES(libzstd INTERFACE lib lib/common)