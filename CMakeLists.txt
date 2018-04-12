# 
# INPUTs:
#
#  IRRLICHT_ROOT: Absolute path to Irrlicht source directory

cmake_minimum_required(VERSION 2.8.11)
project(irrlicht C CXX)

if(NOT IRRLICHT_ROOT)
    message(FATAL_ERROR "IRRLICHT_ROOT is undefined")
endif()

if(NOT EXISTS ${IRRLICHT_ROOT})
    message(FATAL_ERROR "IRRLICHT_ROOT is not found (${IRRLICHT_ROOT})")
endif()

set(src ${IRRLICHT_ROOT}/source/Irrlicht)
set(inc ${IRRLICHT_ROOT}/include)

#
# Read and parse Makefile
#

configure_file(${src}/Makefile ${CMAKE_CURRENT_BINARY_DIR}/Makefile.current)

file(STRINGS ${src}/Makefile makefile)

# Pass1: Concat escaped newline
list(APPEND makefile "\n") # Add a sentinel
set(makefile_lines)
foreach(l ${makefile})
    # We cannot append ${l} directly as CMake variable expansion rule
    set(acc)
    foreach(ln ${l})
        set(acc "${acc}${ln}")
    endforeach()
    list(APPEND makefile_lines ${acc})
endforeach()

# Pass2: Collect Makefile variables
set(slots)
foreach(l ${makefile_lines})
    if(${l} MATCHES "^([^ ]+) = (.*)")
        set(slot ${CMAKE_MATCH_1})
        set(list ${CMAKE_MATCH_2})
        string(REPLACE "\t" "" list ${list})
        separate_arguments(list)
        list(APPEND slots ${slot})
        set(makefile_${slot} ${list})
    endif()
endforeach()

# Pass3: Collect objects
# FIXME: Ignore .mm here.
# FIXME: Allow using system ZLIB/JPEGLIB/LIBPNG/etc.
set(srcs)
foreach(s ${slots})
    foreach(o ${makefile_${s}})
        if(${o} MATCHES "(.*).o$")
            set(basename ${CMAKE_MATCH_1})
            set(cpth "${src}/${basename}.c")
            set(cxxpth "${src}/${basename}.cpp")
            if(EXISTS ${cxxpth})
                list(APPEND srcs ${cxxpth})
            elseif(EXISTS ${cpth})
                list(APPEND srcs ${cpth})
            endif()
        else()
            #message(STATUS "warn: ${o} ignored")
        endif()
    endforeach()
endforeach()

if(IRRLICHT_USE_SDL2_PATCH)
    list(APPEND srcs
        ${src}/CIrrDeviceSDL2.cpp
        ${src}/CIrrDeviceSDL2.h)
endif()

#
# Add STATIC library irrlicht_static
#

# FIXME:
if(${CMAKE_SYSTEM_NAME} STREQUAL "WindowsStore")
    add_definitions(-D_CRT_SECURE_NO_WARNINGS
        -DNO_IRR_COMPILE_WITH_WINDOWS_DEVICE_)
endif()

add_library(irrlicht_static OBJECT ${srcs})
target_include_directories(irrlicht_static PUBLIC ${inc})
target_include_directories(irrlicht_static PRIVATE ${src} 
    # FIXME: Allow using system library
    ${src}/zlib ${src}/jpeglib ${src}/libpng)

