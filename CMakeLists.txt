cmake_minimum_required(VERSION 3.10)
project(c_std)

# Set the C standard
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O3 -march=native -funroll-loops -Wall -Wextra -pedantic -Wno-deprecated-declarations -s")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--enable-auto-import")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--enable-auto-import")

# Set the build directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/dependency/include
)

# Link directories
link_directories(
    ${CMAKE_SOURCE_DIR}/dependency/lib
    ${CMAKE_BINARY_DIR}/build
)

# Add subdirectories for modules
add_subdirectory(string)
add_subdirectory(fmt)
add_subdirectory(encoding)
add_subdirectory(file_io)
add_subdirectory(network)
add_subdirectory(crypto)
# Create the main executable
add_executable(main main.c)

# Link the main executable against all the module libraries and other dependencies
target_link_libraries(main
    stdstring
    fmt
    encoding
    # random
    # map
    # vector
    # stdtime
    # statistics
    # tuple
    # turtle
    # sysinfo
    # stack
    # span
    # queue
    # priority_queue
    # algorithm
    # array
    # bitset
    # cli
    # deque
    # matrix
    # fileio
    # crypto
    # config
    stdcrypto
    # csv
    # date
    # forwardlist
    # list
    network
    # numbers
    # log
    # dir
    # concurrent
    # numeric
    ssl
)

# Copy all necessary DLLs to the build directory
add_custom_command(TARGET main POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
    $<TARGET_FILE:stdstring> $<TARGET_FILE:fmt> $<TARGET_FILE:encoding> 
    $<TARGET_FILE_DIR:main>
)

# Custom command to run the executable
add_custom_target(run
    COMMAND ${CMAKE_COMMAND} -E env "PATH=${CMAKE_RUNTIME_OUTPUT_DIRECTORY};$ENV{PATH}" ./main.exe
    DEPENDS main
    WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
)

# Optional: Add custom command to clean the build directory
add_custom_target(clean_build
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/build
)
