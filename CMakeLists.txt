cmake_minimum_required(VERSION 3.14)

project(ModernRoboticsCpp LANGUAGES CXX)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(MODERN_ROBOTICS_FETCH_DEPENDENCIES "Download missing dependencies during configure" ON)

if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
  set(_modern_robotics_build_tests_default ON)
else()
  set(_modern_robotics_build_tests_default OFF)
endif()
option(MODERN_ROBOTICS_BUILD_TESTS "Build ModernRoboticsCpp tests" ${_modern_robotics_build_tests_default})

include(CMakePackageConfigHelpers)
include(FetchContent)
include(GNUInstallDirs)

find_package(Eigen3 3.3 CONFIG QUIET)

if(NOT TARGET Eigen3::Eigen)
  set(EIGEN3_INCLUDE_DIR "" CACHE PATH "Path to an Eigen3 include directory containing Eigen/Dense")

  if(EIGEN3_INCLUDE_DIR AND EXISTS "${EIGEN3_INCLUDE_DIR}/Eigen/Dense")
    set(_modern_robotics_eigen_include_dir "${EIGEN3_INCLUDE_DIR}")
  elseif(DEFINED ENV{EIGEN3_INCLUDE_DIR} AND EXISTS "$ENV{EIGEN3_INCLUDE_DIR}/Eigen/Dense")
    set(_modern_robotics_eigen_include_dir "$ENV{EIGEN3_INCLUDE_DIR}")
  elseif(MODERN_ROBOTICS_FETCH_DEPENDENCIES)
    set(_modern_robotics_eigen_archive "${CMAKE_BINARY_DIR}/_deps/eigen-3.4.0.tar.gz")
    set(_modern_robotics_eigen_extract_dir "${CMAKE_BINARY_DIR}/_deps/eigen3-extract")
    set(_modern_robotics_eigen_include_dir "${CMAKE_BINARY_DIR}/_deps/eigen3-src")

    if(NOT EXISTS "${_modern_robotics_eigen_include_dir}/Eigen/Dense")
      file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/_deps")
      file(DOWNLOAD
        https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.tar.gz
        "${_modern_robotics_eigen_archive}"
        EXPECTED_HASH SHA256=8586084f71f9bde545ee7fa6d00288b264a2b7ac3607b974e54d13e7162c1c72
        STATUS _modern_robotics_eigen_download_status
      )
      list(GET _modern_robotics_eigen_download_status 0 _modern_robotics_eigen_download_result)
      if(NOT _modern_robotics_eigen_download_result EQUAL 0)
        list(GET _modern_robotics_eigen_download_status 1 _modern_robotics_eigen_download_message)
        message(FATAL_ERROR "Failed to download Eigen3: ${_modern_robotics_eigen_download_message}")
      endif()

      file(REMOVE_RECURSE "${_modern_robotics_eigen_extract_dir}" "${_modern_robotics_eigen_include_dir}")
      file(MAKE_DIRECTORY "${_modern_robotics_eigen_extract_dir}")
      execute_process(
        COMMAND "${CMAKE_COMMAND}" -E tar xzf "${_modern_robotics_eigen_archive}"
        WORKING_DIRECTORY "${_modern_robotics_eigen_extract_dir}"
        RESULT_VARIABLE _modern_robotics_eigen_extract_result
      )
      if(_modern_robotics_eigen_extract_result)
        message(FATAL_ERROR "Failed to extract Eigen3 archive.")
      endif()

      file(GLOB _modern_robotics_eigen_extracted_dirs
        LIST_DIRECTORIES true
        "${_modern_robotics_eigen_extract_dir}/*"
      )
      foreach(_modern_robotics_eigen_extracted_dir IN LISTS _modern_robotics_eigen_extracted_dirs)
        if(EXISTS "${_modern_robotics_eigen_extracted_dir}/Eigen/Dense")
          file(RENAME "${_modern_robotics_eigen_extracted_dir}" "${_modern_robotics_eigen_include_dir}")
        endif()
      endforeach()
      file(REMOVE_RECURSE "${_modern_robotics_eigen_extract_dir}")
    endif()
  else()
    message(FATAL_ERROR
      "Eigen3 was not found. Install Eigen3, set EIGEN3_INCLUDE_DIR to the "
      "directory containing Eigen/Dense, or enable MODERN_ROBOTICS_FETCH_DEPENDENCIES.")
  endif()

  if(_modern_robotics_eigen_include_dir)
    if(NOT EXISTS "${_modern_robotics_eigen_include_dir}/Eigen/Dense")
      message(FATAL_ERROR "Eigen3 include directory does not contain Eigen/Dense: ${_modern_robotics_eigen_include_dir}")
    endif()

    add_library(Eigen3::Eigen INTERFACE IMPORTED)
    set_target_properties(Eigen3::Eigen PROPERTIES
      INTERFACE_INCLUDE_DIRECTORIES "${_modern_robotics_eigen_include_dir}"
    )
  endif()
endif()

if(NOT TARGET Eigen3::Eigen)
  message(FATAL_ERROR "Eigen3::Eigen target was not created.")
endif()

add_library(ModernRoboticsCpp
  src/modern_robotics.cpp
  include/modern_robotics.h
)
add_library(ModernRoboticsCpp::ModernRoboticsCpp ALIAS ModernRoboticsCpp)

target_compile_features(ModernRoboticsCpp PUBLIC cxx_std_11)
target_include_directories(ModernRoboticsCpp
  PUBLIC
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(ModernRoboticsCpp PUBLIC Eigen3::Eigen)

set_target_properties(ModernRoboticsCpp PROPERTIES
  CXX_STANDARD_REQUIRED ON
  CXX_EXTENSIONS OFF
  WINDOWS_EXPORT_ALL_SYMBOLS ON
)

if(MODERN_ROBOTICS_BUILD_TESTS)
  enable_testing()

  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
  set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
  FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
    URL_HASH SHA256=1f357c27ca988c3f7c6b4bf68a9395005ac6761f034046e9dde0896e3aba00e4
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )
  set(_modern_robotics_build_shared_libs "${BUILD_SHARED_LIBS}")
  set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build shared libraries" FORCE)
  FetchContent_MakeAvailable(googletest)
  set(BUILD_SHARED_LIBS "${_modern_robotics_build_shared_libs}" CACHE BOOL "Build shared libraries" FORCE)

  add_executable(lib_test src/lib_test.cpp)
  target_link_libraries(lib_test PRIVATE ModernRoboticsCpp gtest_main)

  include(GoogleTest)
  gtest_discover_tests(lib_test)
endif()

install(TARGETS ModernRoboticsCpp
  EXPORT ModernRoboticsCppTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT ModernRoboticsCppTargets
  NAMESPACE ModernRoboticsCpp::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ModernRoboticsCpp
)

configure_package_config_file(
  cmake/ModernRoboticsCppConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/ModernRoboticsCppConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ModernRoboticsCpp
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/ModernRoboticsCppConfigVersion.cmake
  VERSION 1.0.0
  COMPATIBILITY SameMajorVersion
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/ModernRoboticsCppConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/ModernRoboticsCppConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ModernRoboticsCpp
)
