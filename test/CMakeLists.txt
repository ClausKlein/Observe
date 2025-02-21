cmake_minimum_required(VERSION 3.21...3.28)

project(ObserveTests LANGUAGES CXX)

# ---- Options ----

option(ENABLE_TEST_COVERAGE "Enable test coverage" OFF)
option(TEST_INSTALLED_VERSION "Test the version found by find_package" OFF)

# ---- Dependencies ----

include(../cmake/CPM.cmake)

CPMAddPackage(NAME doctest GITHUB_REPOSITORY onqtam/doctest GIT_TAG v2.4.11)

if(TEST_INSTALLED_VERSION)
  find_package(Observe REQUIRED)
else()
  CPMAddPackage(NAME Observe SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)
endif()

CPMAddPackage(NAME Format.cmake GITHUB_REPOSITORY TheLartians/Format.cmake VERSION 1.8.0)

# ---- Create binary ----

file(GLOB _sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)
add_executable(ObserveTests ${_sources})
target_link_libraries(ObserveTests doctest Observe::Observe)
target_compile_features(ObserveTests INTERFACE cxx_std_17)

# enable compiler warnings
if(NOT TEST_INSTALLED_VERSION)
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(Observe INTERFACE -Wall -pedantic -Wextra -Werror)
  elseif(MSVC)
    target_compile_options(Observe INTERFACE /W4 /WX)
  endif()
endif()

# ---- Add ObserveTests ----

enable_testing()

include(${doctest_SOURCE_DIR}/scripts/cmake/doctest.cmake)
doctest_discover_tests(ObserveTests)

# ---- code coverage ----

if(ENABLE_TEST_COVERAGE AND NOT TEST_INSTALLED_VERSION)
  target_compile_options(Observe INTERFACE -O0 -g -fprofile-arcs -ftest-coverage)
  target_link_options(Observe INTERFACE -fprofile-arcs -ftest-coverage)
endif()
