
enable_language(CUDA)
find_package(CUDAToolkit REQUIRED)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

add_executable( lab4_mm_test
        mm_test.cpp
        ${CMAKE_SOURCE_DIR}/src/gemm.cpp
        ${CMAKE_SOURCE_DIR}/src/gemmRow.cu
        )
target_link_libraries( lab4_mm_test
        PRIVATE
        GTest::gtest_main
        OpenMP::OpenMP_CXX
        OpenCL::OpenCL
        CUDA::cudart  # Link CUDA runtime
        )

include(GoogleTest)
gtest_discover_tests(lab4_mm_test)
