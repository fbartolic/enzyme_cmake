cmake_minimum_required(VERSION 3.12...3.18)
message(STATUS "Using CMake version " ${CMAKE_VERSION})

find_package(Enzyme REQUIRED)

message("LLVM Version: " ${Enzyme_LLVM_VERSION_MAJOR})
message("Found LLVM at: " ${Enzyme_LLVM_BINARY_DIR})

# C and C++ Compilers can only be configured before project(...)
set(CMAKE_C_COMPILER "${Enzyme_LLVM_BINARY_DIR}/bin/clang")
set(CMAKE_CXX_COMPILER "${Enzyme_LLVM_BINARY_DIR}/bin/clang")

project(caustics LANGUAGES CXX)

find_package(Enzyme REQUIRED COMPONENTS ClangEnzyme)
get_target_property(ClangEnzyme ClangEnzyme-${Enzyme_LLVM_VERSION_MAJOR} LOCATION)
message("Found Enzyme at: " ${ClangEnzyme})

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CUDA_STANDARD 14)

find_package(Python COMPONENTS Interpreter Development REQUIRED)
find_package(pybind11 CONFIG REQUIRED)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-experimental-new-pass-manager -fuse-ld=lld -flto --gcc-toolchain=/share/apps/gcc112/ -lstdc++")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fno-experimental-new-pass-manager -fuse-ld=lld -flto")
set(CMAKE_EXE_LINKER_FLAGS "-fuse-ld=lld -flto -Wl,-mllvm=-load=${ClangEnzyme}")
set(CMAKE_SHARED_LINKER_FLAGS "-fuse-ld=lld -flto -Wl,-mllvm=-load=${ClangEnzyme}")

pybind11_add_module(integrate_image_cpu_op ${CMAKE_CURRENT_LIST_DIR}/lib/integrate_image/cpu_ops.cc)
install(TARGETS integrate_image_cpu_op DESTINATION caustics) # caustics is directory
