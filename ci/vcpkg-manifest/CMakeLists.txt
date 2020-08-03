
# CImakeshift test project
# Author: Moritz Beutel


cmake_minimum_required(VERSION 3.16)

# Define the project.
project(CImakeshift-vcpkg-manifest
    LANGUAGES CXX)

# Define build options.
option(BUILD_TESTING "Build tests" OFF)

find_package(CMakeshift 3.8 REQUIRED)
include(CMakeshift/TargetCompileSettings)

# Define targets.
if(BUILD_TESTING)
    enable_testing()

    find_package(fmt REQUIRED)

    add_executable(test-CImakeshift-vcpkg-manifest
        "main.cpp")
    target_compile_features(test-CImakeshift-vcpkg-manifest
        PRIVATE
            cxx_std_17)
    cmakeshift_target_compile_settings(test-CImakeshift-vcpkg-manifest
        PRIVATE
            default)
    target_link_libraries(test-CImakeshift-vcpkg-manifest
        PRIVATE
            fmt::fmt)
    add_test(NAME test-CImakeshift-vcpkg-manifest COMMAND test-CImakeshift-vcpkg-manifest)
    set_property(TEST test-CImakeshift-vcpkg-manifest PROPERTY FAIL_REGULAR_EXPRESSION "Sanitizer")
endif()
