cmake_minimum_required(VERSION 3.1.0)
project(foo)

find_package(Freetype REQUIRED)

# Find Python. Note that the FindPython module is new in CMake 3.12.0, but our
# "minimal" Ubuntu 18.04 builds only have CMake 3.10, so we skip it.
#
if(${CMAKE_VERSION} VERSION_LESS "3.12.0")
    message("Skipping FindPython (CMake < 3.12.0).")
else()
    find_package(Python 3.6 COMPONENTS Interpreter Development)
    if(Python_FOUND)
        execute_process(COMMAND ${Python_EXECUTABLE} -c "import sys; print(sys.prefix, end='')" OUTPUT_VARIABLE VGC_PYTHON_PREFIX)
        execute_process(COMMAND ${Python_EXECUTABLE} -c "import sys; print(sys.exec_prefix, end='')" OUTPUT_VARIABLE VGC_PYTHON_EXEC_PREFIX)
        message(STATUS "Python Version:       ${Python_VERSION}")
        message(STATUS "Python Prefix:        ${PYTHON_PREFIX}")
        message(STATUS "Python Exec Prefix:   ${PYTHON_EXEC_PREFIX}")
        message(STATUS "Python Executable:    ${Python_EXECUTABLE}")
        message(STATUS "Python Libraries:     ${Python_LIBRARIES}")
        message(STATUS "Python Library Dirs:  ${Python_LIBRARY_DIRS}")
        message(STATUS "Python Include Dirs:  ${Python_INCLUDE_DIRS}")
    else()
        message("Python not found.")
    endif()
endif()

# Find Qt5
#
find_package(Qt5 COMPONENTS Core Gui Widgets)
if(Qt5_FOUND)
    get_filename_component(QT_ROOT ${Qt5_DIR}/../../.. ABSOLUTE)
    message(STATUS "Found Qt5: ${QT_ROOT}")
endif()

add_library(foo SHARED src/foo.cpp)
target_include_directories(foo PUBLIC include)
target_link_libraries(foo PUBLIC Freetype::Freetype)
set_target_properties(foo PROPERTIES CXX_VISIBILITY_PRESET hidden)
