cmake_minimum_required(VERSION 3.20)
project(HelloAimIO LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# AimIO exports a target that links against these, so the consumer must load them.
find_package(n88util CONFIG REQUIRED)
find_package(Boost CONFIG REQUIRED COMPONENTS filesystem system)

find_package(AimIO CONFIG REQUIRED)

add_executable(hello main.cpp)
target_link_libraries(hello PRIVATE AimIO::AimIO)

# Explicitly add conda env include dir so "AimIO/AimIO.h" is found
target_include_directories(hello PRIVATE
    $ENV{CONDA_PREFIX}/include
    $ENV{CONDA_PREFIX}/Library/include
)