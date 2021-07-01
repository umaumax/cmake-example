cmake_minimum_required(VERSION 3.5.1)
project(bar)

find_package(foo REQUIRED HINTS "${CMAKE_BINARY_DIR}/foo")

add_executable(bar main.cpp)

# NOTE: include dirs are set automatically
target_link_libraries(bar foo)
