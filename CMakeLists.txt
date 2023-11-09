cmake_minimum_required(VERSION 3.1.0)
project(foo)

# Find Freetype
#
message(STATUS "[Freetype]")
find_package(Freetype REQUIRED)
message(STATUS "") # newline

# Find HarfBuzz
#
# Note: As of CMake 3.18.1 (Jul 2020), there is still no official "FindHarBuzz" module
# On Windows, we use vcpkg which provides a harfbuzz::harfbuzz target
# On Linux/macOS, we are still looking for the best approach
#
message(STATUS "[HarfBuzz]")
find_package(harfbuzz CONFIG QUIET)
if(NOT harfbuzz_FOUND)
    find_package(harfbuzz MODULE REQUIRED)
endif()
get_target_property(HARFBUZZ_INCLUDE_DIRS harfbuzz::harfbuzz INTERFACE_INCLUDE_DIRECTORIES)
if(DEFINED VCPKG_TARGET_TRIPLET AND NOT HARFBUZZ_INCLUDE_DIRS)
    # INTERFACE_INCLUDE_DIRECTORIES is missing in vcpkg <= 2020.06
    # See: https://github.com/microsoft/vcpkg/issues/12847
    set(HARFBUZZ_INCLUDE_DIRS "${_VCPKG_ROOT_DIR}/installed/${VCPKG_TARGET_TRIPLET}/include/harfbuzz")
    set_target_properties(harfbuzz::harfbuzz PROPERTIES INTERFACE_INCLUDE_DIRECTORIES "${HARFBUZZ_INCLUDE_DIRS}")
endif()
if(NOT HARFBUZZ_VERSION_STRING)
    # Sometimes, VERSION_STRING isn't defined, so we do it ourselves. See:
    # https://github.com/microsoft/vcpkg/issues/12848
    # https://github.com/vgc/vgc/pull/413
    if (EXISTS "${HARFBUZZ_INCLUDE_DIRS}/hb-version.h")
        file(READ "${HARFBUZZ_INCLUDE_DIRS}/hb-version.h" _HB_VERSION_H)
        string(REGEX MATCH "define[ \t]+HB_VERSION_MAJOR[ \t]+([0-9]+)" _HARFBUZZ_MAJOR_MATCH "${_HB_VERSION_H}")
        set(HARFBUZZ_VERSION_MAJOR "${CMAKE_MATCH_1}")
        string(REGEX MATCH "define[ \t]+HB_VERSION_MINOR[ \t]+([0-9]+)" _HARFBUZZ_MINOR_MATCH "${_HB_VERSION_H}")
        set(HARFBUZZ_VERSION_MINOR "${CMAKE_MATCH_1}")
        string(REGEX MATCH "define[ \t]+HB_VERSION_MICRO[ \t]+([0-9]+)" _HARFBUZZ_MICRO_MATCH "${_HB_VERSION_H}")
        set(HARFBUZZ_VERSION_MICRO "${CMAKE_MATCH_1}")
        set(HARFBUZZ_VERSION_STRING "${HARFBUZZ_VERSION_MAJOR}.${HARFBUZZ_VERSION_MINOR}.${HARFBUZZ_VERSION_MICRO}")
    endif()
endif()
message(STATUS "Version: ${HARFBUZZ_VERSION_STRING}")
message(STATUS "") # newline

# Find Python. Note that the FindPython module is new in CMake 3.12.0, but our
# "minimal" Ubuntu 18.04 builds only have CMake 3.10, so we skip it.
#
message(STATUS "[Python]")
if(${CMAKE_VERSION} VERSION_LESS "3.12.0")
    message("Skipping FindPython (CMake < 3.12.0).")
else()
    find_package(Python 3.6 COMPONENTS Interpreter Development)
    if(Python_FOUND)
        execute_process(COMMAND ${Python_EXECUTABLE} -c "import sys; print(sys.prefix, end='')" OUTPUT_VARIABLE Python_PREFIX)
        execute_process(COMMAND ${Python_EXECUTABLE} -c "import sys; print(sys.exec_prefix, end='')" OUTPUT_VARIABLE Python_EXEC_PREFIX)
        message(STATUS "Python Version:       ${Python_VERSION}")
        message(STATUS "Python Prefix:        ${Python_PREFIX}")
        message(STATUS "Python Exec Prefix:   ${Python_EXEC_PREFIX}")
        message(STATUS "Python Executable:    ${Python_EXECUTABLE}")
        message(STATUS "Python Libraries:     ${Python_LIBRARIES}")
        message(STATUS "Python Library Dirs:  ${Python_LIBRARY_DIRS}")
        message(STATUS "Python Include Dirs:  ${Python_INCLUDE_DIRS}")
    else()
        message("Python not found.")
    endif()
endif()
message(STATUS "") # newline

# Find Qt5
#
message(STATUS "[Qt]")
find_package(Qt5 COMPONENTS Core Gui Widgets)
if(Qt5_FOUND)
    get_filename_component(QT_ROOT ${Qt5_DIR}/../../.. ABSOLUTE)
    message(STATUS "Found Qt5: ${QT_ROOT}")
else()
    message("Qt not found.")
endif()
message(STATUS "") # newline

add_library(foo SHARED src/foo.cpp)
target_include_directories(foo PUBLIC include)
target_link_libraries(foo PUBLIC Freetype::Freetype harfbuzz::harfbuzz)
set_target_properties(foo PROPERTIES CXX_VISIBILITY_PRESET hidden)
