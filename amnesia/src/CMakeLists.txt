cmake_minimum_required (VERSION 2.8.11)
project(Lux)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR})

OPTION(USERDIR_RESOURCES "Allow using local userdirectory resources" ON)

set(MACOSX_BUNDLE_COPYRIGHT "Copyright © 2010-2013 Frictional Games")

OPTION(IS_APPLE_64 "Apple build in 64 bits" ON)

if(APPLE)
    if(NOT IS_APPLE_64)
        set(CMAKE_OSX_ARCHITECTURES "i386")
    elseif(NOT CMAKE_OSX_ARCHITECTURES)
        # prebuilt dependencies in lib/macosx64 are x86_64 only (runs under Rosetta 2 on Apple Silicon)
        set(CMAKE_OSX_ARCHITECTURES "x86_64")
    endif()
endif()

# silence some annoying warnings
add_definitions(
    -Wno-switch
    -Wno-reorder
    -Wno-unused-variable
    -Wno-unused-function
    # legacy C++98 code: implicit narrowing in brace init is valid there
    -Wno-c++11-narrowing
    -Wno-deprecated-declarations
)

add_subdirectory(../../HPL2/core core2)

if(USERDIR_RESOURCES)
    add_definitions(
        -DUSERDIR_RESOURCES
    )
endif()

set(VERSION "1.3.1")

add_subdirectory(game game)

OPTION(AMNESIA_BUILD_LAUNCHER "Build the graphics quality launcher" OFF)

if(AMNESIA_BUILD_LAUNCHER)
    if(APPLE)
        add_subdirectory(launcher-macosx launcher)
    else()
        add_subdirectory(launcher launcher)
    endif()

    add_custom_target(GameRelease
        DEPENDS Amnesia Launcher
    )
else()
    add_custom_target(GameRelease
        DEPENDS Amnesia
    )
endif()
