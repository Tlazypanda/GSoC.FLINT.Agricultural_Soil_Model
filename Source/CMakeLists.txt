cmake_minimum_required(VERSION 3.10.0)

project(flint.GSoC_Soil VERSION 1.0.0 LANGUAGES CXX)

#turn on using solution folders
set_property( GLOBAL PROPERTY USE_FOLDERS ON)

#turn on parallel builds
add_compile_options($<$<CXX_COMPILER_ID:MSVC>:/MP>)

set(MOJA_FLINT_EXAMPLE_VERSION_MAJOR "1")
set(MOJA_FLINT_EXAMPLE_VERSION_MINOR "0")
set(MOJA_FLINT_EXAMPLE_VERSION_PATCH "0")
set(MOJA_FLINT_EXAMPLE_VERSION "${MOJA_FLINT_EXAMPLE_VERSION_MAJOR}.${MOJA_FLINT_EXAMPLE_VERSION_MINOR}.${MOJA_FLINT_EXAMPLE_VERSION_PATCH}")

# Collect the built libraries and include dirs, the will be used to create the XXX.cmake file
set(MOJA_FLINT_EXAMPLE_COMPONENTS "")

# Allow enabling and disabling components
option(ENABLE_MOJA_FLINT_EXAMPLE_AGRI	    "moja.flint.example.agri"			ON)

# Put the libaries and binaries that get built into directories at the
# top of the build tree rather than in hard-to-find leaf
# directories. This simplifies manual testing and the use of the build
# tree rather than installed Boost libraries.
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# For Debug build types, append a "d" to the library names.
set(CMAKE_DEBUG_POSTFIX "d" CACHE STRING "Set debug library postfix" FORCE)

# set the path for custom cmake scripts
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

option(ENABLE_TESTS "Set to OFF|ON (default is ON) to control build of moja_XXX tests" OFF)
option(RUN_UNIT_TESTS_ON_BUILD "Set to OFF|ON (default is OFF) to control automatic running of moja_XXX tests at build time" OFF)
set( LIB_MODE SHARED )
message(STATUS "Building dynamic libraries")

IF (ENABLE_TESTS)
  include(CTest)
  set(BOOST_TEST_REPORTING_LEVEL "SHORT" CACHE STRING "Boost unit test reporting level")
  set_property(CACHE BOOST_TEST_REPORTING_LEVEL PROPERTY STRINGS "SHORT" "DETAILED")
  add_subdirectory(test)
  ENABLE_TESTING()
  message(STATUS "Building with unittests")
ELSE ()
  message(STATUS "Building without tests")
ENDIF ()

# OS Detection
message(STATUS "The compiler in use is ${CMAKE_CXX_COMPILER}.")
message(STATUS "The compiler flags in use is ${CMAKE_CXX_FLAGS}.")

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("/std:c++17" COMPILER_SUPPORTS_CXX17)
if(COMPILER_SUPPORTS_CXX17)
  message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has C++17 support.")
else()
  message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
endif()

if(CMAKE_SYSTEM MATCHES "Windows")
  add_definitions( -DMOJA_OS_FAMILY_WINDOWS )
  if (CMAKE_C_COMPILER_ID MATCHES "MSVC")
    message(STATUS "XXX: MS Visual Compiler detected")
  endif (CMAKE_C_COMPILER_ID MATCHES "MSVC")
endif(CMAKE_SYSTEM MATCHES "Windows")

if (CMAKE_SYSTEM MATCHES "Linux" )
  add_definitions( -DMOJA_OS_FAMILY_UNIX )
  # Standard 'must be' defines
  #add_definitions( -D_XOPEN_SOURCE=500 -D_REENTRANT -D_THREAD_SAFE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64)
  #set(SYSLIBS  pthread dl rt)
endif(CMAKE_SYSTEM MATCHES "Linux" )

if(ENABLE_MOJA_FLINT_EXAMPLE_AGRI)
add_subdirectory(moja.flint.example.agri)
list(APPEND MOJA_FLINT_EXAMPLE_COMPONENTS "moja.flint.example.agri")
endif()


foreach(component ${MOJA_FLINT_EXAMPLE_COMPONENTS})
message(STATUS "Building: ${component}")
endforeach()


#############################################################
# Uninstall stuff see: http://www.vtk.org/Wiki/CMake_FAQ

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY
)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)

#############################################################
# Enable packaging

set(CPACK_PACKAGE_VERSION_MAJOR "${MOJA_FLINT_EXAMPLE_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${MOJA_FLINT_EXAMPLE_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${MOJA_FLINT_EXAMPLE_VERSION_PATCH}")

set(CPACK_SOURCE_PACKAGE_FILE_NAME
    "${CMAKE_PROJECT_NAME}-${MOJA_FLINT_EXAMPLE_VERSION}"
)

set(CPACK_SOURCE_GENERATOR "TBZ2")
set(CPACK_SOURCE_IGNORE_FILES
    "/.hg/"
    "/.hgignore$"
    "build/"
    "ipch/"
    "/resources/"
    ".sdf$;.suo$;.tss$"
)

include(CPack)
add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)

message(STATUS "CMake ${CMAKE_VERSION} successfully configured ${PROJECT_NAME} using ${CMAKE_GENERATOR} generator")
message(STATUS "Installation target path: ${CMAKE_INSTALL_PREFIX}")

message(STATUS "C_FLAGS:  =${CMAKE_C_FLAGS}")
message(STATUS "CXX_FLAGS:=${CMAKE_CXX_FLAGS}")
