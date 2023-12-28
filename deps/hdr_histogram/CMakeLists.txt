cmake_minimum_required(VERSION 3.10)

# set(CMAKE_C_FLAGS "-std=c99")
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED TRUE)

aux_source_directory (${PROJECT_SOURCE_DIR}/deps/hdr_histogram LUA_SRC)
add_library(hdr_histogram STATIC ${LUA_SRC})
