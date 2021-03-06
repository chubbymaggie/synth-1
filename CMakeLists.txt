# Part of the synth tool -- Copyright (c) Christian Neumüller 2016
# This file is subject to the terms of the MIT License.
# See LICENSE.txt or http://opensource.org/licenses/MIT

cmake_minimum_required(VERSION 2.8)
project(synth)

# http://stackoverflow.com/a/13104057/2128694
macro(prepend_to_all _srcs _path)
    unset(_tmp)
    foreach(src ${${_srcs}})
        list(APPEND _tmp ${_path}${src})
    endforeach()
    set(${_srcs} ${_tmp})
endmacro()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if (MSVC)
    add_definitions("/W4" "/MP")
    add_definitions("-D_SCL_SECURE_NO_WARNINGS")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
    add_definitions("-D__STDC_CONSTANT_MACROS" "-D__STDC_LIMIT_MACROS")
    if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        #set (_cfgfile
        #    "${CMAKE_CURRENT_SOURCE_DIR}/include/apollo/detail/clang_stdlib_config.hpp")
        #add_definitions("-DBOOST_STDLIB_CONFIG=\"${_cfgfile}\"")
        add_definitions("-Weverything")
        add_definitions("-Wno-c++98-compat -Wno-c++98-compat-pedantic")
        add_definitions("-Wno-weak-vtables -Wno-padded")
        add_definitions("-Wno-global-constructors")

        ## LLVM Headers
        #add_definitions("-Wno-conversion")
        #add_definitions("-Wno-sign-conversion")
        #add_definitions("-Wno-shorten-64-to-32")
        #add_definitions("-Wno-old-style-cast")
        #add_definitions("-Wno-unused-parameter")
        #add_definitions("-Wno-documentation-unknown-command")
        #add_definitions("-Wno-shadow")

        set(CMAKE_CXX_FLAGS_DEBUG
            "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=thread,undefined")

    elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        add_definitions("-Wall -Wextra -pedantic")
        add_definitions("-Winit-self -Woverloaded-virtual")
        add_definitions("-Wlogical-op")
        add_definitions("-Wmissing-declarations")
        add_definitions("-Wsign-conversion -Wsign-promo -Wconversion")
        add_definitions("-Wcast-align -Wcast-qual -Wold-style-cast")
        add_definitions("-Wstrict-overflow=5")
        add_definitions("-Wstrict-null-sentinel")
        add_definitions("-Wformat=2")
        add_definitions("-Wmissing-include-dirs")
        add_definitions("-Wdisabled-optimization")
        add_definitions("-Wredundant-decls")
        add_definitions("-Wshadow")
        add_definitions("-Wundef")
        add_definitions("-Wswitch-default")
        add_definitions("-Wnoexcept")

        add_definitions("-Wno-missing-field-initializers")
    endif()
endif()

set (LLVM_VERSION 3.9 CACHE STRING "Version of LLVM/Clang libs to use.")

message(STATUS "LLVM_DIR: $ENV{LLVM_DIR}")
find_path(LIBCLANG_INCLUDE_DIR
    "clang-c/Index.h"
    HINTS "$ENV{LLVM_DIR}/include"
    PATHS
        "/usr/lib/llvm-${LLVM_VERSION}/include/"
        "/usr/include/llvm-${LLVM_VERSION}/llvm/")

find_library(LLVM_LIBRARIES
    HINTS "$ENV{LLVM_DIR}/lib"
    PATHS "/usr/lib/llvm-${LLVM_VERSION}/lib/"
    NAMES clang libclang)


if (WIN32)
    set(Boost_USE_STATIC_LIBS ON)
endif()

# 1.60 is required for Boost.Filesystem's lexically_normal, etc.
find_package(Boost 1.60 REQUIRED filesystem system)
find_package(Threads REQUIRED)

include_directories(SYSTEM ${LIBCLANG_INCLUDE_DIR} ${Boost_INCLUDE_DIRS})

add_subdirectory("src")
