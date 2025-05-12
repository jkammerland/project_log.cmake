cmake_minimum_required(VERSION 3.25)

project(project_log VERSION 1.0.3)

# Option to enable/disable colors
option(PROJECT_LOG_COLORS "Enables colors for STATUS/DEBUG etc" OFF)

# Version checking without preventing inclusion
if(DEFINED PROJECT_LOG_INCLUDED)
  # check version
  if(${PROJECT_VERSION} VERSION_GREATER PROJECT_LOG_INCLUDED)
    message(WARNING "[${PROJECT_NAME}][WARNING] Included ${PROJECT_NAME} [${PROJECT_VERSION}]. Current version is older [${PROJECT_LOG_INCLUDED}]. You may need to update if not forwards compatible.")
  elseif(${PROJECT_VERSION} VERSION_LESS PROJECT_LOG_INCLUDED)
    message(TRACE "[${PROJECT_NAME}][TRACE] Included ${PROJECT_NAME} [${PROJECT_VERSION}]. Current version is newer [${PROJECT_LOG_INCLUDED}]")
  endif()

  get_property(
    PROJECT_LOG_INITIALIZED GLOBAL ""
    PROPERTY PROJECT_LOG_INITIALIZED
    SET)
  if(PROJECT_LOG_INITIALIZED)
    return()
  endif()
else()
  # Mark as included with current version
  set(PROJECT_LOG_INCLUDED
      ${PROJECT_VERSION}
      CACHE INTERNAL "Version of the included project_log module")

  # Only log VERBOSE the first time it's included
  message(VERBOSE "[${PROJECT_NAME}][VERBOSE] Loaded module project_log [${PROJECT_VERSION}]")
endif()

set_property(GLOBAL PROPERTY PROJECT_LOG_INITIALIZED true)

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
    set(COLOR_RESET "${_PROJECT_LOG_ESC_CHAR}[0m")
    set(COLOR_STATUS "${_PROJECT_LOG_ESC_CHAR}[0;32m") # green
    set(COLOR_VERBOSE "${_PROJECT_LOG_ESC_CHAR}[1;32m") # bold green
    set(COLOR_DEBUG "${_PROJECT_LOG_ESC_CHAR}[1;34m") # bold blue
    set(COLOR_TRACE "${_PROJECT_LOG_ESC_CHAR}[0;36m") # cyan
    set(COLOR_WARNING "${_PROJECT_LOG_ESC_CHAR}[1;33m") # bold yellow
    set(COLOR_AUTHOR_WARNING "${_PROJECT_LOG_ESC_CHAR}[0;33m") # yellow
    set(COLOR_DEPRECATION "${_PROJECT_LOG_ESC_CHAR}[1;35m") # bold magenta
    set(COLOR_NOTICE "${_PROJECT_LOG_ESC_CHAR}[0;37m") # light gray
    set(COLOR_ERROR "${_PROJECT_LOG_ESC_CHAR}[1;31m") # bold red
    set(COLOR_FATAL_ERROR "${_PROJECT_LOG_ESC_CHAR}[1;41;37m") # white on red background
  endif()

  if(PROJECT_LOG_COLORS)
    # Select color based on log level
    if(DEFINED COLOR_${level})
      set(level_color "${COLOR_${level}}")
    else()
      set(level_color "${COLOR_RESET}") # Default to reset if level-specific color not found
    endif()

    # Construct the full message with project name and colored level
    set(full_msg "[${_log_context_name}][${level_color}${level}${COLOR_RESET}]${msg}")
  else()
    set(full_msg "[${_log_context_name}][${level}]${msg}")
  endif()

  # Forward the message with the specified log level
  message(${level} "${full_msg}")
endfunction()

# TESTS - Include guard, this must be root dir
if(NOT ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_LIST_DIR})
  return()
else()
  add_subdirectory(tests)
  project_log(STATUS "Configuration complete.")
  project_log(WARNING "A deprecated feature is being used.")
  project_log(DEBUG "Variable X has value:" ${X})
  project_log(VERBOSE "Detailed steps for process Y...")
  # project_log(FATAL_ERROR "Cannot find required resource. Aborting.")
endif()
