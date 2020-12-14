cmake_minimum_required(VERSION 3.1.0)
project(foo)

add_library(foo SHARED src/foo.cpp)
target_include_directories(foo PUBLIC include)
set_target_properties(foo PROPERTIES CXX_VISIBILITY_PRESET hidden)
