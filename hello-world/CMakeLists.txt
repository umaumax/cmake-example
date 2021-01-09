cmake_minimum_required(VERSION 3.5)
project(main)

set(${PROJECT_NAME}_SRCS main.cpp)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
add_definitions(-DXXX)
add_executable(${PROJECT_NAME} ${${PROJECT_NAME}_SRCS})

install(TARGETS ${PROJECT_NAME} DESTINATION bin)
