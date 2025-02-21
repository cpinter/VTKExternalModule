cmake_minimum_required(VERSION 3.8...3.12 FATAL_ERROR)

if (NOT VTK_MODULE_SOURCE_DIR)
  message(FATAL_ERROR "Setting VTK_MODULE_SOURCE_DIR is required to lookup the VTK module sources.")
endif ()
message(STATUS "Setting VTK_MODULE_SOURCE_DIR to ${VTK_MODULE_SOURCE_DIR}")

if (NOT VTK_MODULE_NAME)
  message(FATAL_ERROR "Setting VTK_MODULE_NAME is required to configure the VTK module.")
endif ()
message(STATUS "Setting VTK_MODULE_NAME to ${VTK_MODULE_NAME}")

if (VTK_MODULE_CMAKE_MODULE_PATH)
  message(STATUS "Setting VTK_MODULE_CMAKE_MODULE_PATH to ${VTK_MODULE_CMAKE_MODULE_PATH}")
endif ()

project(vtk${VTK_MODULE_NAME} VERSION 0.1.0)

if (DEFINED VTK_MODULE_ENABLE_VTK_${VTK_MODULE_NAME} AND NOT "${CMAKE_PROJECT_NAME}" STREQUAL "${PROJECT_NAME}")
  # When building as a remote module for VTK 9 (or VTK 8.90+)
  message(FATAL_ERROR "${PROJECT_NAME}: Building as a Remote VTK Module should NOT be done using add_subdirectory()")
  return()
endif ()

find_package(VTK REQUIRED)

if (VTK_VERSION VERSION_LESS 8.90)
  message(FATAL_ERROR "VTK 8.90 or later is required.")
endif ()

option(BUILD_SHARED_LIBS "Build shared libraries." ON)
option(BUILD_TESTING "Build testing" OFF)

if (VTK_MODULE_CMAKE_MODULE_PATH)
  list(INSERT CMAKE_MODULE_PATH 0 ${VTK_MODULE_CMAKE_MODULE_PATH})
endif ()

# Wrap Python option
if (VTK_WRAP_PYTHON)
  option(BUILD_PYTHON_WRAPPERS "Build python wrappers" ON)
else ()
  unset(BUILD_PYTHON_WRAPPERS CACHE)
endif ()

# Wrap Java option
if (VTK_WRAP_JAVA)
  option(BUILD_JAVA_WRAPPERS "Build java wrappers" OFF)
else ()
  unset(BUILD_JAVA_WRAPPERS CACHE)
endif ()

# Defines CMAKE_INSTALL_LIBDIR etcetera
include(GNUInstallDirs)

# Set up our directory structure for output libraries and binaries
# (Note: these are the build locations, not the install locations)
if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
endif ()
if (NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
  if (UNIX)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
  else ()
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
  endif ()
endif ()
if (NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
endif ()

# Search for "vtk.module" files in all subdirectories.
vtk_module_find_modules(vtk_module_files ${VTK_MODULE_SOURCE_DIR})

vtk_module_scan(
  MODULE_FILES     "${vtk_module_files}"
  REQUEST_MODULES  "VTK::${VTK_MODULE_NAME}"
  PROVIDES_MODULES _modules
  ENABLE_TESTS     "${BUILD_TESTING}")

string(TOLOWER ${PROJECT_NAME} _project_name_lc)
vtk_module_build(
  MODULES               ${_modules}
  INSTALL_EXPORT        ${PROJECT_NAME}
  ARCHIVE_DESTINATION   "lib"
  HEADERS_DESTINATION   "include/${_project_name_lc}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
  CMAKE_DESTINATION     "${CMAKE_INSTALL_LIBDIR}/cmake/vtk"
  LICENSE_DESTINATION   "${CMAKE_INSTALL_LIBDIR}/vtk"
  HIERARCHY_DESTINATION "${CMAKE_INSTALL_LIBDIR}/vtk/hierarchy/${CMAKE_PROJECT_NAME}"
  VERSION               "${PROJECT_VERSION}"
  SOVERSION             "1"
  TEST_DATA_TARGET      ${PROJECT_NAME}Data
  USE_EXTERNAL          "1"
  TEST_INPUT_DATA_DIRECTORY   "${_test_data_directory_input}"
  TEST_OUTPUT_DATA_DIRECTORY  "${_test_data_directory_output}")

if (BUILD_PYTHON_WRAPPERS)
  find_package(PythonInterp ${VTK_PYTHON_VERSION} QUIET)

  vtk_module_wrap_python(
    MODULES             ${_modules}
    INSTALL_EXPORT      ${PROJECT_NAME}Python
    PYTHON_PACKAGE      "vtkmodules"
    SOABI               "${Python${VTK_PYTHON_VERSION}_SOABI}"
    CMAKE_DESTINATION   "${CMAKE_INSTALL_LIBDIR}/cmake/vtk"
    LIBRARY_DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    BUILD_STATIC        OFF)
endif ()

if (BUILD_JAVA_WRAPPERS)
  vtk_module_wrap_java(
    MODULES         ${_modules}
    WRAPPED_MODULES _java_wrapped_modules
    JAVA_OUTPUT     "${CMAKE_CURRENT_BINARY_DIR}/CMakeFiles")
endif ()

set(vtk_python_version "")
if (VTK_WRAP_PYTHON)
  set(vtk_python_version "${VTK_PYTHON_VERSION}")
endif ()

# Configure '<projectname>-config.cmake' for a build tree
set(_cmake_dir
    "${CMAKE_CURRENT_SOURCE_DIR}/CMake")
set(_cmake_destination
    "${CMAKE_INSTALL_LIBDIR}/cmake/vtk")
set(_cmake_build_dir
    "${CMAKE_CURRENT_BINARY_DIR}/${_cmake_destination}")

string(TOLOWER ${PROJECT_NAME} _project_name_lc)
configure_file(
  "${_cmake_dir}/vtkmodule-config.cmake.in"
  "${_cmake_build_dir}/${_project_name_lc}-config.cmake"
  @ONLY)

# For convenience, a package is written to the top of the build tree.
file(GENERATE
  OUTPUT  "${CMAKE_BINARY_DIR}/${_project_name_lc}-config.cmake"
  CONTENT "include(\"${_cmake_build_dir}/${_project_name_lc}-config.cmake\")\n")

# Configure '<projectname>-config.cmake' for a install tree

# TODO

