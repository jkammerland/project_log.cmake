project(tests)

add_subdirectory(new_version)

project_log(STATUS "HELLO")

add_subdirectory(old_version)

project_log(STATUS "GOODBYE")
project_log(STATUS "Hello from ${PROJECT_NAME} v${PROJECT_VERSION}")
