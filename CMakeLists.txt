cmake_minimum_required(VERSION 3.22)
project(ydotool C)

set(CMAKE_C_STANDARD 11)

# Include GNU install directory module to detect where to install
# files on Linux/Unix systems (e.g., lib vs lib64)
include(GNUInstallDirs)

execute_process(COMMAND git describe --tags --long --always RESULT_VARIABLE RC_GIT_VER OUTPUT_VARIABLE GIT_VERSION)

if (${RC_GIT_VER} EQUAL 0)
    string(STRIP ${GIT_VERSION} GIT_VERSION)
    message("-- Version: " ${GIT_VERSION})
    add_definitions(-DVERSION=\"${GIT_VERSION}\")
endif()

option(ENABLE_TYPESAFE "Build the 'do' command (natural-language input via TypeSafe; requires libcurl)" OFF)

set(SOURCE_FILES_DAEMON Daemon/ydotoold.c)
set(SOURCE_FILES_CLIENT
    Client/ydotool.c
    Client/tool_click.c
    Client/tool_mousemove.c
    Client/tool_type.c
    Client/tool_key.c
    Client/tool_stdin.c
    Client/keynames.c
    Client/keymap.c)

add_executable(ydotoold ${SOURCE_FILES_DAEMON})
install(TARGETS ydotoold DESTINATION ${CMAKE_INSTALL_BINDIR})

if (ENABLE_TYPESAFE)
    find_package(CURL REQUIRED)
    list(APPEND SOURCE_FILES_CLIENT Client/tool_do.c Client/typesafe.c)
endif ()

add_executable(ydotool ${SOURCE_FILES_CLIENT})
install(TARGETS ydotool DESTINATION ${CMAKE_INSTALL_BINDIR})

if (ENABLE_TYPESAFE)
    target_link_libraries(ydotool PRIVATE CURL::libcurl)
    target_compile_definitions(ydotool PRIVATE HAVE_TYPESAFE=1)
endif ()

add_subdirectory(Daemon)
add_subdirectory(manpage)
