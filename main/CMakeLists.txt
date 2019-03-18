cmake_minimum_required(VERSION 3.5)
project(main)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../hoge
                   ${CMAKE_CURRENT_BINARY_DIR}/hoge)
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../fuga
                   ${CMAKE_CURRENT_BINARY_DIR}/fuga)
endif()

set(${PROJECT_NAME}_SRCS main.cpp)

set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
add_definitions(-DXXX)
add_executable(${PROJECT_NAME} ${${PROJECT_NAME}_SRCS})

target_include_directories(${PROJECT_NAME}
                           PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/../hoge
                                   ${CMAKE_CURRENT_SOURCE_DIR}/../fuga)
link_directories(${CMAKE_BINARY_DIR}/hoge ${CMAKE_BINARY_DIR}/fuga)
target_link_libraries(${PROJECT_NAME} hoge fuga)

install(TARGETS ${PROJECT_NAME} DESTINATION bin)
