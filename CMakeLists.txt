# Created by SwiftWare Lab on 9/24.
# CE 4SP4 - High Performance Programming
# Copyright (c) 2024 SwiftWare Lab

cmake_minimum_required(VERSION 3.22)

option(PROFILING_ENABLED "Enables Modules for Profiling, needs LIBFPM4" OFF)
option(OPENMP "Enables OpenMP" ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native -mavx")
include(FetchContent)

FetchContent_Declare(
        argparse
        GIT_REPOSITORY https://github.com/p-ranav/argparse.git
)
FetchContent_MakeAvailable(argparse)


include(FetchContent)
FetchContent_Declare(
        googlebenchmark
        GIT_REPOSITORY https://github.com/google/benchmark.git
        GIT_TAG v1.9.0
)
# enable BENCHMARK_DOWNLOAD_DEPENDENCIES
set(BENCHMARK_DOWNLOAD_DEPENDENCIES ON)
if (PROFILING_ENABLED)
    # enable PFM
    set(BENCHMARK_ENABLE_LIBPFM ON)
endif()



FetchContent_MakeAvailable(googlebenchmark)


project(lab4 C CXX )

#check CUDA version
find_package(CUDA OPTIONAL_COMPONENTS)
find_package(CUDAToolkit OPTIONAL_COMPONENTS)
# enable_language(CUDA)
# find_package(CUDAToolkit REQUIRED)
if (CUDA_FOUND AND CUDAToolkit_FOUND)
    message(STATUS "CUDA found: ${CUDA_VERSION}")
    add_definitions(-D__CUDA__)
    enable_language(CUDA)

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")

    set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -G --ptxas-options=-v")
    set(CMAKE_CUDA_STANDARD_REQUIRED TRUE)
    set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
    set(CMAKE_CUDA_ARCHITECTURES 75 80 86)
    message(STATUS "CUDA_ARCHITECTURES: ${CMAKE_CUDA_ARCHITECTURES}")

    include_directories(${CUDA_INCLUDE_DIRS})
    include_directories(${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})
    message(STATUS "CUDA_INCLUDE_DIRS: ${CUDA_INCLUDE_DIRS}")
    message(STATUS "CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES: ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}")
    #    add_executable(vec_mul_vec
    #            ${CMAKE_SOURCE_DIR}/src/vec_mul.cu
    #    )


else()
    message(STATUS "CUDA not found, proceeding without CUDA support")
    # Set the project languages to only CXX if CUDA is not found
    set(PROJECT_LANGUAGES CXX)
endif()


find_package(OpenCL REQUIRED)
find_package(OpenMP REQUIRED)

include_directories(
        ${CMAKE_SOURCE_DIR}/include
        # OpenCL headers
        ${OpenCL_INCLUDE_DIRS}
)

set(CMAKE_CXX_STANDARD 17)


if (CUDA_FOUND AND CUDAToolkit_FOUND)
    add_executable(matmul
        ${CMAKE_SOURCE_DIR}/matmul_driver.cpp
        ${CMAKE_SOURCE_DIR}/src/gemm.cpp
        ${CMAKE_SOURCE_DIR}/src/gemmRow.cu
    )
else()
add_executable(matmul
        ${CMAKE_SOURCE_DIR}/matmul_driver.cpp
        ${CMAKE_SOURCE_DIR}/src/gemm.cpp
)

endif()


target_include_directories(matmul PUBLIC ${OpenCL_INCLUDE_DIRS})
target_link_directories(matmul PUBLIC ${OpenCL_LIBRARY})


target_link_libraries(matmul
        benchmark::benchmark
        OpenMP::OpenMP_CXX
	    OpenCL::OpenCL
)


add_subdirectory(test)