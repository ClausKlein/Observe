cmake_minimum_required(VERSION 3.21...3.28)

# ---- Project ----

# Note: update this to your new project's name and version
project(Observe VERSION 3.0 LANGUAGES CXX)

# ---- Include guards ----

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR
            "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# --- Import tools ----

include(cmake/tools.cmake)

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(cmake/CPM.cmake)

# PackageProject.cmake will be used to make our target installable
CPMAddPackage(NAME PackageProject.cmake GITHUB_REPOSITORY TheLartians/PackageProject.cmake VERSION 1.11.2)

# ---- Add source files ----
# Note: globbing sources is considered bad practice as CMake's generators may not detect new files
# automatically. Keep that in mind when changing files, or explicitly mention them here.
file(GLOB_RECURSE _headers CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")

# ---- Create library ----

# TODO: not used? CK
# add_library(ObserveHeaders EXCLUDE_FROM_ALL ${_headers})
# set_target_properties(ObserveHeaders PROPERTIES LINKER_LANGUAGE CXX)

add_library(Observe INTERFACE)
target_compile_features(Observe INTERFACE cxx_std_17)

# being a cross-platform target, we enforce enforce standards conformance on MSVC
target_compile_options(Observe INTERFACE "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/permissive>")

target_include_directories(
  Observe INTERFACE "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
                    "$<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>"
)

# for run-clang-tidy
add_executable(example example/example.cpp)
target_link_libraries(example Observe)

option(ENABLE_WARNING_AS_ERRORS "Enable pedantic warnings" PROJECT_IS_TOP_LEVEL)
if(ENABLE_WARNING_AS_ERRORS)
  include(cmake/WarningsAsErrors.cmake)
endif()

if(CMAKE_SKIP_INSTALL_RULES)
  return()
endif()

# ---- Create an installable target ----
# this allows users to install and find the library via `find_package()`.
include(cmake/AddUninstallTarget.cmake)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  NAMESPACE ${PROJECT_NAME}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}-${PROJECT_VERSION}
  DEPENDENCIES ""
)

include(CPack)
