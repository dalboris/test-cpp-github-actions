cmake_minimum_required(VERSION 3.1.0)
project(foo)

find_package(Freetype REQUIRED)
find_package(Qt5 COMPONENTS Core Gui Widgets)

add_library(foo SHARED src/foo.cpp)
target_include_directories(foo PUBLIC include)
target_link_libraries(foo PUBLIC Freetype::Freetype)
set_target_properties(foo PROPERTIES CXX_VISIBILITY_PRESET hidden)
