cmake_minimum_required(VERSION 3.9)
project(geographiclib_vendor)

find_package(ament_cmake REQUIRED)

set(PACKAGE_VERSION "1.0.0")

macro(build_geographiclib)
  set(extra_cmake_args)

  list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=ON)
  list(APPEND extra_cmake_args -DBUILD_TESTING=OFF)
  list(APPEND extra_cmake_args -DCMAKE_C_FLAGS=${CMAKE_C_FLAGS})
  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
  endif()

  include(ExternalProject)
  ExternalProject_Add(geographiclib-1.5.2
    PREFIX geographiclib-1.5.2
    GIT_REPOSITORY https://github.com/geographiclib/geographiclib
    GIT_TAG v1.52
    TIMEOUT 600
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/geographiclib_install
      -DINSTALL_INCLUDE_DIR:STRING=include/${PROJECT_NAME}
      ${extra_cmake_args}
    PATCH_COMMAND
      ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> git apply -p1 --ignore-space-change --whitespace=nowarn
        ${CMAKE_CURRENT_SOURCE_DIR}/0001-geographiclib.patch
  )
endmacro()

find_package(geographiclib QUIET)
if(NOT geographiclib_FOUND)
  build_geographiclib()
else()
  message(STATUS "Found geographiclib ${geographiclib_VERSION} in path ${geographiclib_CONFIG}")
endif()

ament_package()
