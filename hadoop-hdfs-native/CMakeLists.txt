#
# CMakeLists.txt
#
#
# The MIT License
#
# Copyright (c) 2018 Omics Data Automation, Inc.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(HDFSWrapper)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

if (WIN32)
  message(FATAL "No support for WIN32")
endif()

function(create_hdfs_objs LIBNAME)
    add_library(hdfs_OBJECTS OBJECT ${ARGN})
endfunction(create_hdfs_objs)

# Check to see if our compiler and linker support the __thread attribute.
# On Linux and some other operating systems, this is a more efficient 
# alternative to POSIX thread local storage. 
INCLUDE(CheckCSourceCompiles)
CHECK_C_SOURCE_COMPILES("int main(void) { static __thread int i = 0; return 0; }" HAVE_BETTER_TLS)

# Check to see if we have Intel SSE intrinsics.
CHECK_C_SOURCE_COMPILES("#include <emmintrin.h>\nint main(void) { __m128d sum0 = _mm_set_pd(0.0,0.0); return 0; }" HAVE_INTEL_SSE_INTRINSICS)

# Check if we need to link dl library to get dlopen.
# dlopen on Linux is in separate library but on FreeBSD its in libc
INCLUDE(CheckLibraryExists)
CHECK_LIBRARY_EXISTS(dl dlopen "" NEED_LINK_DL)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
find_package(JNI REQUIRED)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Wall -O2")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_REENTRANT -D_GNU_SOURCE")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
set(OS_DIR main/native/libhdfs/os/posix)

add_definitions(-DLIBHDFS_DLL_EXPORT)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}
    ${JNI_INCLUDE_DIRS}
    main/native
    main/native/libhdfs
    ${OS_DIR}
)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_BINARY_DIR}/config.h)

create_hdfs_objs(hdfs
    main/native/libhdfs/exception.c
    main/native/libhdfs/jni_helper.c
    main/native/libhdfs/hdfs.c
    main/native/libhdfs/common/htable.c
    ${OS_DIR}/mutexes.c
    ${OS_DIR}/thread_local_storage.c
)

set(OBJS_DIR ${CMAKE_BINARY_DIR}/hdfs_objs)

set(
    FIND_ARGUMENTS
    "${CMAKE_BINARY_DIR}/CMakeFiles -name *.c.o -exec cp {} ${OBJS_DIR} \\;"
)

install(DIRECTORY DESTINATION ${OBJS_DIR}) 
install(CODE "execute_process(COMMAND find ${FIND_ARGUMENTS})")

