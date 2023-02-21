cmake_minimum_required(VERSION 3.1.0)
project(foo)

find_package(Freetype REQUIRED)

find_package(Python 3.6 COMPONENTS Interpreter Development REQUIRED)

find_package(Qt5 COMPONENTS Core Gui Widgets)
if(Qt5_FOUND)
    get_filename_component(QT_ROOT ${Qt5_DIR}/../../.. ABSOLUTE)
    message(STATUS "Found Qt5: ${QT_ROOT}")
endif()

add_library(foo SHARED src/foo.cpp)
target_include_directories(foo PUBLIC include)
target_link_libraries(foo PUBLIC Freetype::Freetype)
set_target_properties(foo PROPERTIES CXX_VISIBILITY_PRESET hidden)
