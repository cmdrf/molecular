cmake_minimum_required(VERSION 3.8)
project(molecular)

set(CMAKE_CXX_STANDARD 14)

find_package(Eigen3)
find_package(OpenGL)
find_package(Threads)

add_subdirectory(3rdparty)
add_subdirectory(molecular-util)
add_subdirectory(molecular-meshfile)
add_subdirectory(molecular-programgenerator)
add_subdirectory(molecular-gfx)
