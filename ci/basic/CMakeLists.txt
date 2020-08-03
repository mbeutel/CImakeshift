
# CImakeshift test project
# Author: Moritz Beutel


cmake_minimum_required(VERSION 3.16)

# Define the project.
project(CImakeshift-basic
    LANGUAGES CXX)

# Define build options.
option(BUILD_TESTING "Build tests" OFF)
option(BUILD_TESTING_CUDA "Build CUDA tests" OFF)

find_package(CMakeshift 3.8 REQUIRED)
include(CMakeshift/TargetCompileSettings)

# Define targets.
if(BUILD_TESTING)
    enable_testing()

    find_package(gsl-lite 0.36 REQUIRED)

    add_executable(test-CImakeshift-basic
        "main.cpp")
    target_compile_features(test-CImakeshift-basic
        PRIVATE
            cxx_std_14)
    cmakeshift_target_compile_settings(test-CImakeshift-basic
        PRIVATE
            default)
    target_link_libraries(test-CImakeshift-basic
        PRIVATE
            gsl::gsl-lite-v1)
    add_test(NAME test-CImakeshift-basic COMMAND test-CImakeshift-basic)
    set_property(TEST test-CImakeshift-basic PROPERTY FAIL_REGULAR_EXPRESSION "Sanitizer")

    if(BUILD_TESTING_CUDA)
        enable_language(CUDA)

        add_executable(test-CImakeshift-basic-CUDA
            "main.cu")
        target_compile_features(test-CImakeshift-basic-CUDA
            PRIVATE
                cxx_std_14)
        cmakeshift_target_compile_settings(test-CImakeshift-basic-CUDA
            PRIVATE
                default)
        target_link_libraries(test-CImakeshift-basic-CUDA
            PRIVATE
                gsl::gsl-lite-v1)
        add_test(NAME test-CImakeshift-basic-CUDA COMMAND test-CImakeshift-basic-CUDA)
        set_property(TEST test-CImakeshift-basic-CUDA PROPERTY FAIL_REGULAR_EXPRESSION "Sanitizer")
    endif()
endif()
