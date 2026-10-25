cmake_minimum_required(VERSION 3.10)
project(sigclick C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Strict warnings
add_compile_options(-Wall -Wextra -Wpedantic)

# Optional sanitizers for debug
option(ENABLE_SANITIZERS "Enable AddressSanitizer and UndefinedBehaviorSanitizer" OFF)
if(ENABLE_SANITIZERS)
    add_compile_options(-fsanitize=address,undefined -g)
    add_link_options(-fsanitize=address,undefined)
endif()

include_directories(${CMAKE_SOURCE_DIR}/include)

add_executable(sigclick
    src/main.c
)

# Math library not strictly needed, but safe
target_link_libraries(sigclick m)

install(TARGETS sigclick DESTINATION bin)
