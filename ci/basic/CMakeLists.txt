
# CImakeshift test project
# Author: Moritz Beutel


cmake_minimum_required(VERSION 3.16)

# Define the project.
project(CImakeshift-basic
    LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
include(TargetCompileSettings)

# Define build options.
option(BUILD_TESTING "Build tests" OFF)
option(BUILD_TESTING_CUDA "Build CUDA tests" OFF)

# Define targets.
if(BUILD_TESTING)
    enable_testing()

    find_package(gsl-lite 0.36 REQUIRED)

    add_executable(test-CImakeshift-basic
        "main.cpp"
    )
    target_compile_features(test-CImakeshift-basic
        PRIVATE
            cxx_std_14
    )
    cmakeshift_target_compile_settings(test-CImakeshift-basic)
    target_link_libraries(test-CImakeshift-basic
        PRIVATE
            gsl::gsl-lite-v1
    )
    add_test(NAME test-CImakeshift-basic COMMAND test-CImakeshift-basic)

    if(BUILD_TESTING_CUDA)
        enable_language(CUDA)

        add_executable(test-CImakeshift-basic-CUDA
            "main.cu"
        )
        target_compile_features(test-CImakeshift-basic-CUDA
            PRIVATE
                cxx_std_14
        )
        cmakeshift_target_compile_settings(test-CImakeshift-basic-CUDA)
        target_link_libraries(test-CImakeshift-basic-CUDA
            PRIVATE
                gsl::gsl-lite-v1
        )
        add_test(NAME test-CImakeshift-basic-CUDA COMMAND test-CImakeshift-basic-CUDA)
    endif()
endif()
