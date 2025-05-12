# Project Log CMake Module

A simple CMake module to enhance logging within your CMake scripts. It provides a `project_log` function that prefixes messages with the current project name and log level, and optionally adds color to the output for better readability. This aims to solve the problem where you don't know where the log came from when a function can be invoked from multiple places.

## Features

*   **Standardized Logging**: Prepends `[PROJECT_NAME][LEVEL]` to all log messages.
*   **Standard CMake Levels**: Supports all standard CMake message levels (`FATAL_ERROR`, `SEND_ERROR`, `WARNING`, `AUTHOR_WARNING`, `DEPRECATION`, `NOTICE`, `STATUS`, `VERBOSE`, `DEBUG`, `TRACE`). They work in the same way as message(Level ...)
*   **Color-Coded Output**: Optionally uses ANSI colors for different log levels (can be toggled with the `PROJECT_LOG_COLORS` option).
*   **Version Tracking**: Warns if different versions of the module are included within the same CMake build.

## Integration with FetchContent

You can integrate `project_log` into your CMake project using `FetchContent`. Add the following to your main `CMakeLists.txt`:

```cmake
include(FetchContent)

FetchContent_Declare(
  project_log
  GIT_REPOSITORY https://github.com/jkammerland/project_log.cmake.git
  GIT_TAG v1.0.3  # or branch/commit                                              
)

# To enable colors, set this *before* FetchContent_MakeAvailable
# option(PROJECT_LOG_COLORS "Enable log colors" ON) # Or set via cmake -DPROJECT_LOG_COLORS=ON

FetchContent_MakeAvailable(project_log)
# The project_log function is now available.
```
or cpmaddpackage
```cmake
cpmaddpackage("gh:jkammerland/project_log.cmake@1.0.3")
# The project_log function is now available.
```

The `project_log` module's `CMakeLists.txt` is designed to be included directly.

## Usage

### Calling the `project_log` function

After making the module available, use the `project_log` function similarly to CMake's built-in `message()` command:

```cmake
# Assuming your project is named "MyAwesomeApp" via project(MyAwesomeApp)

project_log(STATUS "Configuration complete.")
project_log(WARNING "A deprecated feature is being used.")
project_log(DEBUG "Variable X has value:" ${X})
project_log(VERBOSE "Detailed steps for process Y...")
project_log(FATAL_ERROR "Cannot find required resource. Aborting.")
```

### Example Output
```
[MyAwesomeApp][STATUS] Configuration complete.
[MyAwesomeApp][WARNING] A deprecated feature is being used.
[MyAwesomeApp][DEBUG] Variable X has value: some_value
[MyAwesomeApp][VERBOSE] Detailed steps for process Y...
CMake Warning at tests/new_version/CMakeLists.txt:122 (message):
  [MyAwesomeApp][WARNING] A deprecated feature is being used.
Call Stack (most recent call first):
  CMakeLists.txt:132 (project_log)


CMake Error at tests/new_version/CMakeLists.txt:122 (message):
  [MyAwesomeApp][FATAL_ERROR] Cannot find required resource.  Aborting.
Call Stack (most recent call first):
  CMakeLists.txt:135 (project_log)
```

### Enabling/Disabling Colors

Colors are **OFF** by default. To enable them, set the `PROJECT_LOG_COLORS` CMake option to `ON`. This must be done **before** the `project_log` module is processed (e.g., before `FetchContent_MakeAvailable(project_log)`).

You can set this option:
1.  When invoking CMake:
    ```bash
    cmake -DPROJECT_LOG_COLORS=ON --log-level=TRACE .. # or any other log-level
    ```
2.  In your `CMakeLists.txt` (before fetching/including `project_log`):
    ```cmake
    set(PROJECT_LOG_COLORS ON)
    ```

## Log Levels

The `project_log` function accepts the standard CMake log levels:
*   `FATAL_ERROR`
*   `SEND_ERROR`
*   `WARNING`
*   `AUTHOR_WARNING`
*   `DEPRECATION`
*   `NOTICE` (or if no level is specified to `message()`)
*   `STATUS`
*   `VERBOSE`
*   `DEBUG`
*   `TRACE`

Each level will be distinctly colored if `PROJECT_LOG_COLORS` is enabled.