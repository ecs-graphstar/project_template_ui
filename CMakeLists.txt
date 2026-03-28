# Default Paphos project starter
# flecs/PyTorch with AOT Inductor pipeline

cmake_minimum_required(VERSION 3.18)
project(project)

find_package(Git REQUIRED)

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/deps/glfw/CMakeLists.txt")
    message(STATUS "Initial clone detected. Fetching submodules shallowly...")
    execute_process(
        COMMAND ${GIT_EXECUTABLE} -c submodule.fetchJobs=8 
                submodule update --init --recursive --depth 1
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
endif()

add_executable(${PROJECT_NAME} src/main.cpp)
include_directories("src")

add_definitions(-DTRACY_ENABLE)
add_subdirectory("deps/tracy")

# Flecs
include_directories("deps/flecs/include")
set(BUILD_SHARED_LIBS ON CACHE BOOL "Build Flecs as a shared library" FORCE)
add_subdirectory("deps/flecs")

# Flecs custom build configuration - must match defines in main.cpp
target_compile_definitions(flecs PUBLIC
    FLECS_CUSTOM_BUILD
    FLECS_ALERTS
    FLECS_APP
    FLECS_CPP
    FLECS_DOC
    FLECS_JSON
    FLECS_HTTP
    FLECS_LOG
    FLECS_META
    FLECS_METRICS
    FLECS_MODULE
    FLECS_OS_API_IMPL
    FLECS_PERF_TRACE
    FLECS_PIPELINE
    FLECS_REST
    FLECS_PARSER
    FLECS_QUERY_DSL
    FLECS_SCRIPT
    FLECS_SYSTEM
    FLECS_STATS
    FLECS_TIMER
    FLECS_UNITS
)

# GLFW
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
add_subdirectory("deps/glfw")

# GLAD
add_library(glad STATIC
    deps/glad/src/glad.c
)
target_include_directories(glad PUBLIC deps/glad/include)

# NanoVG
add_library(nanovg STATIC
    deps/nanovg/src/nanovg.c
)
target_include_directories(nanovg PUBLIC deps/nanovg/src)
target_compile_definitions(nanovg PUBLIC NANOVG_GL2_IMPLEMENTATION)

target_link_libraries(${PROJECT_NAME} flecs glfw glad nanovg Tracy::TracyClient)

set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 17)
