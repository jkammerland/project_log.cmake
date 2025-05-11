cmake_minimum_required(VERSION 3.25)

project(project_log VERSION 1.0.0)

# Option to enable/disable colors
option(PROJECT_LOG_COLORS "Enables colors for STATUS/DEBUG etc" OFF)

# Option to enable/disable tests
option(PROJECT_LOG_ENABLE_TESTS "Enable tests for the project_log module" OFF)

# Version checking without preventing inclusion
if(DEFINED PROJECT_LOG_INCLUDED)
  # If already included, check version
  if(NOT ${PROJECT_VERSION} VERSION_EQUAL PROJECT_LOG_INCLUDED)
    message(WARNING "project_log module version mismatch: included version ${PROJECT_LOG_INCLUDED}, current version ${PROJECT_VERSION}.")
    if(${PROJECT_VERSION} VERSION_GREATER PROJECT_LOG_INCLUDED)
      message(STATUS "Must override version to ${PROJECT_VERSION}. Might not be forwards compatible.")
      set(PROJECT_LOG_INCLUDED
          ${PROJECT_VERSION}
          CACHE INTERNAL "Version of the included project_log module")
    else()
      message(STATUS "Will not override to older version ${PROJECT_VERSION}. This function is backwards compatible up to new major versions.")
      return()
    endif()
  else()

  endif()
else()
  # Mark as included with current version
  set(PROJECT_LOG_INCLUDED
      ${PROJECT_VERSION}
      CACHE INTERNAL "Version of the included project_log module")

  # Only log the first time it's included
  if(PROJECT_NAME)
    set(_project_log_context_name "${PROJECT_NAME}")
  else()
    set(_project_log_context_name "CMake") # Default if project() not yet called
  endif()
  # Use basic message since project_log is not yet defined
  message(VERBOSE "[${_project_log_context_name}] Loaded project_log module v${PROJECT_VERSION}")
endif()

# Define ANSI escape character once
string(ASCII 27 _PROJECT_LOG_ESC_CHAR)

# ~~~
# project_log(Level Arg1 Arg2 ...)
#
#
# Level is the same as standard cmake, and is summerized in the following table:
#
# | Level | Description |
# |-------|-------------|
# | `FATAL_ERROR`      | Stops processing & generation. Makes cmake exit with non-zero code.
# | `SEND_ERROR`       | Reports error, continues processing but skips generation.
# | `WARNING`          | Shows warning message, continues processing.
# | `AUTHOR_WARNING`   | Developer warning, continues processing.
# | `DEPRECATION`      | Shows deprecation notice if `CMAKE_ERROR/WARN_DEPRECATED` enabled.
# | `NOTICE` or (none) | Important message printed to stderr.
# | `STATUS`           | Key information users might need, ideally brief.
# | `VERBOSE`          | Additional details for interested users.
# | `DEBUG`            | Implementation details for project developers.
# | `TRACE`            | Temporary fine-grained messages about internal details.
# ~~~
function(project_log level)
  # Determine project context name
  if(PROJECT_NAME)
    set(_log_context_name "${PROJECT_NAME}")
  else()
    set(_log_context_name "CMake") # Default context if PROJECT_NAME is not set
  endif()

  # Collect all the arguments after 'level' into a single message
  string(JOIN " " _message_content ${ARGN})
  set(msg "") # Initialize msg
  if(NOT _message_content STREQUAL "")
    set(msg " ${_message_content}") # Prepend space if there's content, to match original format
  endif()

  # Define ANSI color codes for different log levels (using CMake-compatible escapes)
  if(WIN32)
    # Windows terminals might not support ANSI colors by default Setting them to empty effectively disables them for Windows here.
    set(COLOR_RESET "")
    set(COLOR_STATUS "")
    set(COLOR_VERBOSE "")
    set(COLOR_DEBUG "")
    set(COLOR_TRACE "")
    set(COLOR_WARNING "")
    set(COLOR_AUTHOR_WARNING "")
    set(COLOR_DEPRECATION "")
    set(COLOR_NOTICE "")
    set(COLOR_ERROR "")
    set(COLOR_FATAL_ERROR "")
  else()
    set(ESC "${_PROJECT_LOG_ESC_CHAR}") # Use pre-defined escape character
    set(COLOR_RESET "${ESC}[0m")
    set(COLOR_STATUS "${ESC}[0;32m") # green
    set(COLOR_VERBOSE "${ESC}[1;32m") # bold green
    set(COLOR_DEBUG "${ESC}[1;34m") # bold blue
    set(COLOR_TRACE "${ESC}[0;36m") # cyan
    set(COLOR_WARNING "${ESC}[1;33m") # bold yellow
    set(COLOR_AUTHOR_WARNING "${ESC}[0;33m") # yellow
    set(COLOR_DEPRECATION "${ESC}[1;35m") # bold magenta
    set(COLOR_NOTICE "${ESC}[0;37m") # light gray
    set(COLOR_ERROR "${ESC}[1;31m") # bold red
    set(COLOR_FATAL_ERROR "${ESC}[1;41;37m") # white on red background
  endif()

  if(PROJECT_LOG_COLORS)
    # Select color based on log level
    if(DEFINED COLOR_${level})
      set(level_color "${COLOR_${level}}")
    else()
      set(level_color "${COLOR_RESET}") # Default to reset if level-specific color not found
    endif()

    # Construct the full message with project name and colored level
    set(full_msg "[${_log_context_name}[${level_color}${level}${COLOR_RESET}]${msg}")
  else()
    set(full_msg "[${_log_context_name}][${level}]${msg}")
  endif()

  # Forward the message with the specified log level
  message(${level} "${full_msg}")
endfunction()

if(PROJECT_LOG_ENABLE_TESTS)
  add_subdirectory(tests)
endif()
