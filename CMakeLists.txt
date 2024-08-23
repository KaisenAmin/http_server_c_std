cmake_minimum_required(VERSION 3.10)
project(c_std)

# Set the C standard
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O3 -march=native -funroll-loops -Wall -Wextra -pedantic -Wno-deprecated-declarations -s")

# Platform-specific linker flags
if (WIN32)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--enable-auto-import")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--enable-auto-import")
endif()

# Set the build directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/build)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/dependency/include
)

# Link directories
link_directories(
    if (WIN32)
        ${CMAKE_SOURCE_DIR}/dependency/lib
    endif()
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
if (WIN32)
    target_link_libraries(main
        stdstring
        fmt
        encoding
        stdcrypto
        network
        ssl
        crypto
    )
else()
    find_package(OpenSSL REQUIRED)
    target_link_libraries(main
        stdstring
        fmt
        encoding
        stdcrypto
        network
        OpenSSL::SSL
        OpenSSL::Crypto
        pthread
        dl
    )
endif()

# Custom command to run the executable
if (WIN32)
    add_custom_target(run
        COMMAND ${CMAKE_COMMAND} -E env "PATH=${CMAKE_RUNTIME_OUTPUT_DIRECTORY};$ENV{PATH}" ./main.exe
        DEPENDS main
        WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
    )
else()
    add_custom_target(run
        COMMAND ./main
        DEPENDS main
        WORKING_DIRECTORY ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}
    )
endif()

# Optional: Add custom command to clean the build directory
add_custom_target(clean_build
    COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/build
)
