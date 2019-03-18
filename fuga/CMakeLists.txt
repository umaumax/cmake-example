cmake_minimum_required(VERSION 3.5)
project(fuga)

set(${PROJECT_NAME}_SRCS fuga.cpp)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
add_library(${PROJECT_NAME} ${${PROJECT_NAME}_SRCS})

install(TARGETS ${PROJECT_NAME} DESTINATION lib)
