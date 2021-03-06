cmake_minimum_required(VERSION 3.0.0)
project(ISLE)

add_definitions(-DLINUX)
# Use -DCILK above if you are certain, Mixing Cilk + Intel MKL/OpenMP causes problems

# Compiler options
set(CXX g++)

# CXX Config
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -DDEBUG -O0 -fsanitize=address -fsanitize=leak -fsanitize=undefined")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Ofast -march=native -mtune=native")
add_compile_options(-std=c++14 -Wall ${CONFIG_FLAGS} -fopenmp -Wfatal-errors)

#MKL Config
set(INTEL_ROOT /opt/intel/compilers_and_libraries/linux)
set(MKL_ROOT ${INTEL_ROOT}/mkl)
add_definitions(-DMKL_ILP64)
add_compile_options(-m64 -Wl,--no-as-needed)
include_directories(include ${INTEL_ROOT}/include ${MKL_ROOT}/include)
link_directories(${INTEL_ROOT}/lib/intel64 ${MKL_ROOT}/lib/intel64)
#link_libraries(mkl_intel_ilp64 mkl_sequential mkl_core iomp5 pthread m dl cilkrts)
link_libraries(mkl_intel_ilp64 mkl_intel_thread mkl_core iomp5 pthread m dl)

# Other config
add_compile_options(${OPTIM_CXXFLAGS})

# Generate library
add_library(ISLE STATIC src/trainer.cpp src/utils.cpp src/denseMatrix.cpp src/sparseMatrix.cpp src/logger.cpp src/infer.cpp)
target_include_directories(ISLE BEFORE PUBLIC ${CMAKE_SOURCE_DIR} armadillo-8.400.0/include spectra-master block-ks)
link_libraries(ISLE)

# Generate drivers
add_executable(ISLETrain drivers/ISLETrain.cpp)
add_executable(ISLEInfer drivers/ISLEInfer.cpp)

target_link_libraries(ISLETrain ISLE)
target_link_libraries(ISLEInfer ISLE)
