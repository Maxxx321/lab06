cmake_minimum_required(VERSION 3.4)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)



add_library(formatter_lib STATIC ../formatter_lib/formatter.cpp)
add_library(formatter_ex_lib STATIC ../formatter_ex_lib/formatter_ex.cpp)
add_library(solver_lib STATIC ../solver_lib/solver.cpp)

include_directories(	../formatter_lib
			../formatter_ex_lib
			../solver_lib)


add_executable(solver equation.cpp)

target_link_libraries(solver 	solver_lib 
				formatter_ex_lib 
				formatter_lib)