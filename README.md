# Project Log CMake Module

A simple CMake module to enhance logging within your CMake scripts. It provides a `project_log` function that prefixes messages with the current project name and log level, and optionally adds color to the output for better readability.

## Features

*   **Standardized Logging**: Prepends `[PROJECT_NAME][LEVEL]` to all log messages.
*   **Standard CMake Levels**: Supports all standard CMake message levels (`FATAL_ERROR`, `SEND_ERROR`, `WARNING`, `AUTHOR_WARNING`, `DEPRECATION`, `NOTICE`, `STATUS`, `VERBOSE`, `DEBUG`, `TRACE`).
*   **Color-Coded Output**: Optionally uses ANSI colors for different log levels (can be toggled with the `PROJECT_LOG_COLORS` option).
*   **Version Tracking**: Includes its own version and warns if different versions of the module are included within the same CMake build.
*   **Load Indication**: Logs a message when it's first included, showing its version.

## Integration with FetchContent

You can integrate `project_log` into your CMake project using `FetchContent`. Add the following to your main `CMakeLists.txt`:

```cmake
include(FetchContent)

FetchContent_Declare(
  project_log
  GIT_REPOSITORY https://your.git.repository/path/to/project_log.git 
  GIT_TAG 1.0.0  # Master                                              
)

# To enable colors, set this *before* FetchContent_MakeAvailable
# option(PROJECT_LOG_COLORS "Enable log colors" ON) # Or set via cmake -DPROJECT_LOG_COLORS=ON

FetchContent_MakeAvailable(project_log)
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

**Example Output (without colors, `PROJECT_LOG_COLORS` is `OFF`):**
```
[MyAwesomeApp[STATUS] Configuration complete.
[MyAwesomeApp][WARNING] A deprecated feature is being used.
[MyAwesomeApp][DEBUG Variable X has value: some_value
[MyAwesomeApp][VERBOSE] Detailed steps for process Y...
-- Configuring incomplete, errors occurred!
CMake Error at CMakeLists.txt:XX (project_log):
  [MyAwesomeApp][FATAL_ERROR] Cannot find required resource. Aborting.
```

**Example Output (with colors, `PROJECT_LOG_COLORS` is `ON` and terminal supports ANSI):**
```
[MyAwesomeApp][<green_bold>STATUS<reset>] Configuration complete.
[MyAwesomeApp][<yellow>WARNING<reset>] A deprecated feature is being used.
[MyAwesomeApp][<blue_bold>DEBUG<reset>] Variable X has value: some_value
[MyAwesomeApp][<green>VERBOSE<reset>] Detailed steps for process Y...
-- Configuring incomplete, errors occurred!
CMake Error at CMakeLists.txt:XX (project_log):
  [MyAwesomeApp][<white_on_red_bg>FATAL_ERROR<reset>] Cannot find required resource. Aborting.
```

### Enabling/Disabling Colors

Colors are **OFF** by default. To enable them, set the `PROJECT_LOG_COLORS` CMake option to `ON`. This must be done **before** the `project_log` module is processed (e.g., before `FetchContent_MakeAvailable(project_log)`).

You can set this option:
1.  When invoking CMake:
    ```bash
    cmake -DPROJECT_LOG_COLORS=ON ..
    ```
2.  In your `CMakeLists.txt` (before fetching/including `project_log`):
    ```cmake
    set(PROJECT_LOG_COLORS ON CACHE BOOL "Enable log colors for project_log" FORCE)
    # or
    # option(PROJECT_LOG_COLORS "Enable log colors for project_log" ON)
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