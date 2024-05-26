cmake_minimum_required(VERSION 3.4)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(hello_world hello_world.cpp)

add_library(formatter_lib STATIC ../formatter_lib/formatter.cpp)
add_library(formatter_ex_lib STATIC ../formatter_ex_lib/formatter_ex.cpp)


target_include_directories(formatter_lib PUBLIC ../formatter_lib)
target_include_directories(formatter_ex_lib PUBLIC ../formatter_ex_lib ../formatter_lib)
target_include_directories(hello_world PUBLIC ../formatter_ex_lib ../formatter_lib)

target_link_libraries(hello_world formatter_ex_lib formatter_lib)