cmake_policy(SET CMP0091 NEW)
project(StardustDevEnvironment)
cmake_minimum_required(VERSION 3.14)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

IF(MSVC)
    set(CMAKE_CXX_FLAGS "/std:c++17")
    add_definitions(-DWITHOUT_POSIX)
ENDIF(MSVC)

set(SIMULATE true)

IF(NOT SIMULATE)
    include_directories(SYSTEM
        ${CMAKE_SOURCE_DIR}/3rdparty/BWAPILIB/include
        ${CMAKE_SOURCE_DIR}/3rdparty
    )
    add_subdirectory(${CMAKE_SOURCE_DIR}/3rdparty/BWAPILIB)
ELSE(NOT SIMULATE)
    #[set(OPENBW_DIR ${CMAKE_SOURCE_DIR}/3rdparty/openbw/openbw)]#
    add_subdirectory(${CMAKE_SOURCE_DIR}/test)
    add_subdirectory(${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi)
ENDIF(NOT SIMULATE)

add_subdirectory(3rdparty/zstd/build/cmake)
add_subdirectory(src)
