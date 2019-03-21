cmake_minimum_required(VERSION 3.5.2)

project(libfoo)

set(CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE ON)

add_library(foo SHARED foo.cpp)

set_target_properties(foo PROPERTIES PUBLIC_HEADER foo/foo.hpp)

# target_link_libraries(foo PUBLIC foo_depend_lib)

# NOTE: module mode: Findfoo
# NOTE: config mode: foo-config
set(export_name foo-config)
install(TARGETS foo
        EXPORT ${export_name}
        LIBRARY DESTINATION lib
        INCLUDES
        DESTINATION include
        PUBLIC_HEADER DESTINATION include/foo)

install(EXPORT ${export_name}
        FILE ${export_name}.cmake
        DESTINATION share/cmake/foo/
        EXPORT_LINK_INTERFACE_LIBRARIES)

# NOTE: for other cmake files in same build tree
export(TARGETS foo FILE ${export_name}.cmake EXPORT_LINK_INTERFACE_LIBRARIES)
# NOTE: touch dummy file for other cmake files can find
# NOTE: below command is run at prebuild
file(TOUCH "${CMAKE_CURRENT_BINARY_DIR}/${export_name}.cmake")
