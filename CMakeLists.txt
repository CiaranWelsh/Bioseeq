cmake_minimum_required(VERSION 3.16)

set(BIOSEEQ_VERSION_MAJOR 1)
set(BIOSEEQ_VERSION_MINOR 11)
set(BIOSEEQ_VERSION_MICRO 0)
set(VERSION "${BIOSEEQ_VERSION_MAJOR}.${BIOSEEQ_VERSION_MINOR}.${BIOSEEQ_VERSION_MICRO}")

project(bioseeq VERSION ${VERSION})




set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_VERBOSE_MAKEFILE ON)

# put all runtime targets (exe and dll) into bin
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)

# put libraries into lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

# archives
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lib)

if (MSVC)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}" "/NODEFAULTLIB")
endif ()

########################################################
#   dependencies
#

find_package(BZip2 REQUIRED)
find_package(LibLZMA REQUIRED)
find_package(ZLIB REQUIRED)
find_package(CURL REQUIRED)
find_package(lz4 CONFIG REQUIRED)
# note, it seems that on windows and linux, openssl is a requirement for HMAC
# while on apple we instead need common crypto (a note from from htslib cmake script)
find_package(OpenSSL)

# This was akward. Installed on vcpkg then went to
# D:\vcpkg\installed\x64-windows-static\share\pthread
# to figure out what variables pthreads imports.
# Looks like PThreads4W_LIBRARY_RELEASE holds our release library
if (WIN32)
    find_package(pthreads CONFIG REQUIRED)
else()
    # on linux this works fine but doesn't work on windows
    find_package(Threads REQUIRED)
endif ()


set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH}" "${CMAKE_CURRENT_SOURCE_DIR}/cmake")


# find bcftools
find_path(
        SAMTOOLS_INSTALL_PREFIX
        NAMES
            lib/${CMAKE_SHARED_LIBRARY_PREFIX}samtools${CMAKE_SHARED_LIBRARY_SUFFIX}
            include/samtools/samtools.h
        PATHS
            "${CMAKE_CURRENT_SOURCE_DIR}"
            "${OneDirectoryAboveWhereWeAre}"
            "${DIRECTORIES_IN_POTENTIAL_SAMTOOLS_LOCATION}"
            "${SAMTOOLS_INSTALL_PREFIX}" # alias
)
if (NOT SAMTOOLS_INSTALL_PREFIX)
    message(WARNING "Could not automatically locate\
 path to samtools. Please provide the path to the install\
 tree for samtools to
-DSAMTOOLS_INSTALL_PREFIX=/path/to/samtools/install/prefix
 If you add this directory to PATH or install\
 samtools in a system directory (/usr/local or C:/Program Files) it will\
be found automatically. ")
endif ()

message(STATUS "SAMTOOLS_INSTALL_PREFIX: ${SAMTOOLS_INSTALL_PREFIX}/lib/cmake/samtools")
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH}" "${SAMTOOLS_INSTALL_PREFIX}/lib/cmake/samtools")


find_package(samtools CONFIG REQUIRED COMPONENT STATIC_LIBRARY)


add_library(bioseeq STATIC src/)
































