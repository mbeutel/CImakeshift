
# CImakeshift test project
# Author: Moritz Beutel


cmake_minimum_required(VERSION 3.16)

# Define the project.
project(CImakeshift
    LANGUAGES CXX)

# Define build options.
option(BUILD_TESTING "Build tests" OFF)
option(BUILD_TESTING_CUDA "Build CUDA tests" OFF)

find_package(CMakeshift 3.8 REQUIRED)

# Define targets.
if(BUILD_TESTING)
    enable_testing()

    find_package(Catch2 2.11 REQUIRED)

    add_executable(test-CImakeshift
        "main.cpp")
    target_compile_features(test-CImakeshift
        PRIVATE
            cxx_std_14)
    target_compile_settings(test-CImakeshift
        PRIVATE
            default)
    target_link_libraries(test-CImakeshift
        PRIVATE
            Catch2::Catch2)
    add_test(NAME test-CImakeshift COMMAND test-CImakeshift)
    set_property(TEST test-CImakeshift PROPERTY FAIL_REGULAR_EXPRESSION "Sanitizer")

    if(BUILD_TESTING_CUDA)
        enable_language(CUDA)

        add_executable(test-CImakeshift-CUDA
            "main.cu")
        target_compile_features(test-CImakeshift-CUDA
            PRIVATE
                cxx_std_14)
        target_compile_settings(test-CImakeshift-CUDA
            PRIVATE
                default)
        target_link_libraries(test-CImakeshift-CUDA
            PRIVATE
                Catch2::Catch2)
        add_test(NAME test-CImakeshift-CUDA COMMAND test-CImakeshift-CUDA)
        set_property(TEST test-CImakeshift-CUDA PROPERTY FAIL_REGULAR_EXPRESSION "Sanitizer")
    endif()
endif()
