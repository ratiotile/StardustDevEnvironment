cmake_minimum_required(VERSION 3.14)
project(tests)

set(gtest_force_shared_crt ON CACHE BOOL "")
option(gtest_force_shared_crt "" ON)
add_subdirectory(3rdparty/googletest)

add_subdirectory(3rdparty/opponents/Steamhammer)
add_subdirectory(3rdparty/opponents/Locutus)

include_directories(${CMAKE_SOURCE_DIR}/3rdparty/googletest/include)

include_directories(SYSTEM
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/Util/Source
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/BWAPI/openbw
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/BWAPI/openbw/BWAPI
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/BWAPI/Source
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/BWAPICore
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/OpenBWData
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/Shared
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/openbw
)

set(TEST_INCLUDES
        ${PROJECT_SOURCE_DIR}
        )
include_directories(${TEST_INCLUDES})

file(GLOB_RECURSE SRC_FILES ${PROJECT_SOURCE_DIR} *.cpp)
list(FILTER SRC_FILES EXCLUDE REGEX ".*3rdparty.*")

add_executable(tests
        $<TARGET_OBJECTS:BWAPIObj>
        $<TARGET_OBJECTS:BWAPICore>
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/OpenBWData/BW/BWData.cpp
        ${SRC_FILES}
        )

set_source_files_properties(
        $<TARGET_OBJECTS:BWAPIObj>
        $<TARGET_OBJECTS:BWAPICore>
        ${CMAKE_SOURCE_DIR}/3rdparty/openbw/bwapi/bwapi/OpenBWData/BW/BWData.cpp
        PROPERTIES
        #[COMPILE_FLAGS "-w"]#
)

if(CMAKE_BUILD_TYPE MATCHES Debug)
    target_compile_definitions(tests PUBLIC DEBUG=1)
endif()

target_sources(tests
    PRIVATE ${CMAKE_SOURCE_DIR}/src/Instrumentation/Log.cpp
)

target_link_libraries(tests gtest gtest_main BWAPILIB DemoAIModule Steamhammer Locutus DemoAIModule)
#[target_compile_options(tests PRIVATE -Wall -Werror)]#

file(COPY ${CMAKE_SOURCE_DIR}/test/3rdparty/maps DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bwapi-data/AI)
file(COPY ${CMAKE_SOURCE_DIR}/test/3rdparty/bwta DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bwapi-data/AI)
file(COPY ${CMAKE_SOURCE_DIR}/test/3rdparty/opponents/Steamhammer/Steamhammer_2.4.2.json DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bwapi-data/AI)
file(COPY ${CMAKE_SOURCE_DIR}/test/3rdparty/opponents/Locutus/Locutus.json DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bwapi-data/AI)

IF(WIN32)
    MESSAGE(STATUS "Setting #define WIN32")
    ADD_COMPILE_DEFINITIONS(
            WIN32       #[Timer.hpp keeps trying to include sys/time.h, maybe cmake is not #defining WIN32]#
            NOMINMAX    #[windows.h max() macro causes compile errors]#
    )
    target_compile_definitions(Locutus PRIVATE WIN32 NOMINMAX)
    target_compile_definitions(Steamhammer PRIVATE WIN32 NOMINMAX)
ENDIF(WIN32)