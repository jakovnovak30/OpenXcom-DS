cmake_minimum_required ( VERSION 3.5 )

# specify cross compilers and tools
set(CMAKE_C_COMPILER /sbin/arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER /sbin/arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER /sbin/arm-none-eabi-gcc)
set(CMAKE_AR /sbin/arm-none-eabi-ar)
set(CMAKE_OBJCOPY /sbin/arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP /sbin/arm-none-eabi-objdump)
set(SIZE /sbin/arm-none-eabi-size)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

set(DS_FLAGS "-O3 -mcpu=arm9tdmi")
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} ${DS_FLAGS}")
set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} ${DS_FLAGS}")

project ( OpenXcom )
set ( CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules" )
include(BuildType)
include(GNUInstallDirs)

# For yaml-cpp
set (CMAKE_CXX_STANDARD 11)
# use std=c++11 rather than std=gnu++11
set (CMAKE_CXX_EXTENSIONS ON)
set (CMAKE_C_EXTENSIONS ON)

option ( DEV_BUILD "Development Build. Disable this for release builds" ON )
option ( BUILD_PACKAGE "Prepares build for creation of a package with CPack" ON )
option ( ENABLE_WARNING "Always show warnings (even for release builds)" OFF )
option ( FATAL_WARNING "Treat warnings as errors" OFF )
option ( ENABLE_CLANG_ANALYSIS "When building with clang, enable the static analyzer" OFF )
option ( CHECK_CCACHE "Check if ccache is installed and use it" OFF )
set ( MSVC_WARNING_LEVEL 3 CACHE STRING "Visual Studio warning levels" )
option ( FORCE_INSTALL_DATA_TO_BIN "Force installation of data to binary directory" OFF )
set ( DATADIR "" CACHE STRING "Where to search for datafiles" )
set ( OPENXCOM_VERSION_STRING "" CACHE STRING "Version string (after x.x)" )

if ( CHECK_CCACHE )
  find_program( CCACHE_PROGRAM ccache )
  if( NOT CCACHE_PROGRAM )
    message ( "CCACHE requested but not found on the system." )
  else ()
    set_property( GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}" )
    set_property( GLOBAL PROPERTY RULE_LAUNCH_LINK "${CCACHE_PROGRAM}" )
    message ( "found CCACHE (${CCACHE_PROGRAM})" )
  endif()
endif ()

# Read version number
set ( file "${CMAKE_SOURCE_DIR}/src/version.h" )
file ( READ ${file} lines )
string ( REGEX MATCH "[.]*OPENXCOM_VERSION_SHORT \"([0-9]).([0-9])" version_line "${lines}" )
set ( CPACK_PACKAGE_VERSION_MAJOR ${CMAKE_MATCH_1} )
set ( CPACK_PACKAGE_VERSION_MINOR ${CMAKE_MATCH_2} )
set ( CPACK_PACKAGE_VERSION_PATCH "" )

if ( "${OPENXCOM_VERSION_STRING}" STREQUAL "" )
  find_package ( Git )
  if ( GIT_FOUND )
    message("git found: ${GIT_EXECUTABLE}")
    execute_process ( COMMAND ${GIT_EXECUTABLE} describe --dirty
      WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
      OUTPUT_VARIABLE git_describe_out
      ERROR_VARIABLE git_describe_error
      RESULT_VARIABLE git_describe_result
      )
    string ( REGEX MATCH "([a-z|0-9|.]*)-([0-9]*)-g([a-z|0-9]*)([-|a-z]*)" git_commit "${git_describe_out}" )
    set ( git_tag ${CMAKE_MATCH_1} )
    set ( git_nb_commit ${CMAKE_MATCH_2} )
    set ( git_commit ${CMAKE_MATCH_3} )
    set ( git_dirty ${CMAKE_MATCH_4} )
    set ( OPENXCOM_VERSION_STRING ".${git_commit}${git_dirty}" )
  endif()
endif()

add_definitions( -DGIT_BUILD=1 )

configure_file("${CMAKE_SOURCE_DIR}/src/git_version.h.in" "${CMAKE_CURRENT_BINARY_DIR}/git_version.h" )
include_directories ( "${CMAKE_CURRENT_BINARY_DIR}" )


if ( DEV_BUILD )
  # Append the commit to version number
  set ( CPACK_PACKAGE_VERSION_PATCH "${git_commit}${git_dirty}" )
  set ( CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}" )
else ()
  set ( CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_NSIS_PACKAGE_NAME}" )
  set ( CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}" )
endif ()

if ( BUILD_PACKAGE )
  if ( NOT DEV_BUILD )
    string ( LENGTH "${git_dirty}" is_dirty )
    if ( ${is_dirty} GREATER 0 )
      message ( FATAL_ERROR "Release package must be built from a clean tree" )
    endif ()
    if ( "${CMAKE_BUILD_TYPE}" STREQUAL "Debug" )
      message ( FATAL_ERROR "Release package can't be built from a debug build" )
    endif ()
  endif ( )

  if ( NOT CPACK_GENERATOR )
    set ( CPACK_GENERATOR "TXZ" )
  endif ()
  if ( NOT CPACK_SOURCE_GENERATOR )
    set ( CPACK_SOURCE_GENERATOR "TXZ" )
  endif ()

  set ( CPACK_PACKAGE_VENDOR "The OpenXcom project" )
  set ( CPACK_PACKAGE_DESCRIPTION_SUMMARY "Open-source clone of UFO: Enemy Unknown" )
  set ( CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/cmake/modules/Description.txt" )
  set ( CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md" )
  if ( NOT APPLE )
    set ( CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE.txt" )
  endif ()
  set ( CPACK_PACKAGE_CONTACT "The OpenXcom project (http://www.openxcom.org)" )

  include ( LinuxDEB )
  include ( LinuxRPM )
  include ( nsis )
  include ( apple )
  include ( CPack )
  message ( "version:${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}"
)
endif()

if ( NOT WIN32 )
  install(FILES "${CMAKE_SOURCE_DIR}/res/linux/openxcom.desktop"
    DESTINATION "${CMAKE_INSTALL_FULL_DATAROOTDIR}/applications")
  install(FILES "${CMAKE_SOURCE_DIR}/res/linux/icons/openxcom_48x48.png"
    DESTINATION "${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/48x48/apps" RENAME openxcom.png)
  install(FILES "${CMAKE_SOURCE_DIR}/res/linux/icons/openxcom_128x128.png"
    DESTINATION "${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/128x128/apps" RENAME openxcom.png)
  install(FILES "${CMAKE_SOURCE_DIR}/res/linux/icons/openxcom.svg"
    DESTINATION "${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/scalable/apps")
endif ()


add_subdirectory(libs/yaml-cpp)

add_library(SDL STATIC IMPORTED)

set_target_properties(SDL PROPERTIES
    IMPORTED_LOCATION
        ${CMAKE_SOURCE_DIR}/libs/SDL-nds/lib/libSDL.a

    INTERFACE_INCLUDE_DIRECTORIES
        ${CMAKE_SOURCE_DIR}/libs/SDL-nds/include
)

add_subdirectory ( docs )
add_subdirectory ( src )
