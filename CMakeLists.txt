cmake_minimum_required(VERSION 2.8.3)

project(reflexxes_type2)

find_package(catkin REQUIRED)


include(ExternalProject)

set(RML_SRC_DIR "${CMAKE_BINARY_DIR}/upstream_reflexxes-prefix/src/upstream_reflexxes")
set(RML_HEADER_DIR "${RML_SRC_DIR}/include/")
set(RML_RELEASE_LIBRARY_DIR "${RML_SRC_DIR}/Linux/x64/release/lib/shared/")
set(RML_DEBUG_LIBRARY_DIR "${RML_SRC_DIR}/Linux/x64/debug/lib/shared/")

#To avoid problem on early check of existance of this directory
file(MAKE_DIRECTORY ${RML_HEADER_DIR})

ExternalProject_Add(
  upstream_reflexxes
  GIT_REPOSITORY https://github.com/code-iai/RMLTypeII.git
  #GIT_TAG xx
  BINARY_DIR upstream_reflexxes-prefix/src/upstream_reflexxes/Linux
  CONFIGURE_COMMAND ""
  BUILD_COMMAND make clean64 all64
  INSTALL_COMMAND ls "${RML_HEADER_DIR}"
  #Currently the upstream Makefile has no installation target
  #INSTALL_DIR ${CATKIN_DEVEL_PREFIX}
  )

install(
  DIRECTORY ${RML_HEADER_DIR}
  DESTINATION include
  FILES_MATCHING PATTERN "*.h"
  )

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    install(DIRECTORY ${RML_DEBUG_LIBRARY_DIR}
    DESTINATION lib
    FILES_MATCHING PATTERN "*${CMAKE_SHARED_LIBRARY_SUFFIX}"
    )
else()
    install(DIRECTORY ${RML_RELEASE_LIBRARY_DIR}
    DESTINATION lib
    FILES_MATCHING PATTERN "*${CMAKE_SHARED_LIBRARY_SUFFIX}"
    )
endif()

configure_file(reflexxes_type2-config.cmake.in
  ${PROJECT_BINARY_DIR}/reflexxes_type2-config.cmake @ONLY)
configure_file(reflexxes_type2-config-version.cmake.in
  ${PROJECT_BINARY_DIR}/reflexxes_type2-config-version.cmake @ONLY)

install(FILES ${PROJECT_BINARY_DIR}/reflexxes_type2-config.cmake DESTINATION share/reflexxes_type2/cmake)
install(FILES ${PROJECT_BINARY_DIR}/reflexxes_type2-config-version.cmake DESTINATION share/reflexxes_type2/cmake)
install(FILES package.xml DESTINATION share/reflexxes_type2)

add_library(ReflexxesTypeII SHARED IMPORTED)
set_target_properties(ReflexxesTypeII PROPERTIES IMPORTED_LOCATION ${RML_RELEASE_LIBRARY_DIR}/libReflexxesTypeII.so)
add_dependencies(ReflexxesTypeII upstream_reflexxes)

catkin_package(
  LIBRARIES ReflexxesTypeII
  INCLUDE_DIRS ${RML_HEADER_DIR}
)

#MESSAGE(WARNING "INCLUDE_DIRS: ${RML_HEADER_DIR}")

