cmake_minimum_required(VERSION 3.25)

project(project_log VERSION 1.99.4)

# Option to enable/disable colors
option(PROJECT_LOG_COLORS "Enables colors for STATUS/DEBUG etc" OFF)

set(PROJECT_INCLUDE_GUARD_VERSION "1.2.0")
get_property(
  CPM_INITIALIZED GLOBAL
  PROPERTY CPM_INITIALIZED
  SET)
if(CPM_INITIALIZED)
  cpmaddpackage("gh:jkammerland/project_include_guard.cmake@${PROJECT_INCLUDE_GUARD_VERSION}")
else()
  include(FetchContent)
  FetchContent_Declare(
    project_include_guard
    GIT_REPOSITORY https://github.com/jkammerland/project_include_guard.cmake.git
    GIT_TAG "v${PROJECT_INCLUDE_GUARD_VERSION}" # or branch/commit
  )
  FetchContent_MakeAvailable(project_include_guard)
endif(CPM_INITIALIZED)
project_include_guard()

# TESTS - Include guard, this must be root dir
option(${PROJECT_NAME}_BUILD_TESTS "${PROJECT_NAME}_BUILD_TESTS" OFF)
if(${PROJECT_NAME}_BUILD_TESTS)
  add_subdirectory(tests)
  project_log(STATUS "Configuration complete.")
  project_log(WARNING "A deprecated feature is being used.")
  project_log(DEBUG "Variable X has value:" ${X})
  project_log(VERBOSE "Detailed steps for process Y...")
  # project_log(FATAL_ERROR "Cannot find required resource. Aborting.")
endif()
