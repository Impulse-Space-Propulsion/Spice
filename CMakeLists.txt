cmake_minimum_required(VERSION 3.4)
project(Spice LANGUAGES C CXX)

# Include all *.c files from the library
file(GLOB SPICE_COMMON_SOURCE ${PROJECT_SOURCE_DIR}/src/common/*.c)
set(INCLUDE_PATH "include")
file(GLOB SPICE_HEADERS include/*.h)

if (MSVC)
    file(GLOB SPICE_PLATFORM_SOURCE ${PROJECT_SOURCE_DIR}/src/msvc/*.c)
    set(INCLUDE_PATH "include/msvc")
elseif (APPLE)
    file(GLOB SPICE_PLATFORM_SOURCE ${PROJECT_SOURCE_DIR}/src/mac/*.c)
    set(INCLUDE_PATH "include/mac")
elseif (CYGWIN)
    file(GLOB SPICE_PLATFORM_SOURCE ${PROJECT_SOURCE_DIR}/src/cygwin/*.c)
    set(INCLUDE_PATH "include/cygwin")
elseif (MINGW)
    # Untested
    file(GLOB SPICE_PLATFORM_SOURCE ${PROJECT_SOURCE_DIR}/src/linux/*.c)
    set(INCLUDE_PATH "include/linux")
elseif (UNIX)
    file(GLOB SPICE_PLATFORM_SOURCE ${PROJECT_SOURCE_DIR}/src/linux/*.c)
    set(INCLUDE_PATH "include/linux")
else()
    #Die if no match found
    fatal_error("Environment not matched to one of: MSVC, APPLE, CYGWIN, MINGW, UNIX")
endif ()

option(SPICE_BUILD_SHARED_LIBRARY "Build SPICE library as shared library" OFF)
message("Spice Platform Sources: " ${SPICE_PLATFORM_SOURCES})

if (SPICE_BUILD_SHARED_LIBRARY)
    add_library(Spice SHARED ${SPICE_COMMON_SOURCE} ${SPICE_PLATFORM_SOURCE})
else ()
    add_library(Spice STATIC ${SPICE_COMMON_SOURCE} ${SPICE_PLATFORM_SOURCE})
endif ()

set_target_properties(Spice 
    PROPERTIES 
        WINDOWS_EXPORT_ALL_SYMBOLS TRUE
        POSITION_INDEPENDENT_CODE TRUE)

target_include_directories(Spice
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/${INCLUDE_PATH}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${INCLUDE_PATH}>
    $<INSTALL_INTERFACE:include>
  PRIVATE
  )


if (WIN32)
    # Our compile definitions on Windows
    # We are not manually setting the optimization level, but leave that to the Debug/Release flags
    target_compile_definitions(Spice PRIVATE 
        _COMPLEX_DEFINED
        MSDOS
        OMIT_BLANK_CC
        NON_ANSI_STDIO
    )
    target_compile_options(Spice PRIVATE /MP)

    set_target_properties(Spice PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
else ()
    target_compile_definitions(Spice PRIVATE NON_ANSI_STDIO)
    target_compile_options(Spice PRIVATE -ansi)
endif ()

if (MSVC)
    target_compile_options(Spice PRIVATE /W0)
else ()
    target_compile_options(Spice PRIVATE -w)
endif ()

# 'make install' to the correct locations (provided by GNUInstallDirs).
install(TARGETS Spice EXPORT SpiceConfig
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR})  # This is for Windows
install(DIRECTORY ${INCLUDE_PATH} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${SPICE_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# This makes the project importable from the install directory
# Put config file in per-project dir (name MUST match), can also
# just go into 'cmake'.
install(
  EXPORT SpiceConfig 
  DESTINATION lib/cmake/Spice
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/libFC/
)
export(
  TARGETS Spice
  FILE "${CMAKE_CURRENT_BINARY_DIR}/SpiceTargets.cmake"
  NAMESPACE Spice::
)
install(
  EXPORT SpiceConfig
  DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake/Spice/
  NAMESPACE Spice::
)

