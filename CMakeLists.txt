cmake_minimum_required(VERSION 3.2)
project(hapfuse VERSION 3.0)

find_package(Boost COMPONENTS program_options iostreams filesystem REQUIRED)
if (Boost_FOUND)
  include_directories(${BOOST_INCLUDE_DIRS})
endif()


find_package (Threads REQUIRED)
find_package (ZLIB REQUIRED)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON) #...is required...
set(CMAKE_CXX_EXTENSIONS OFF) #...without compiler extensions like gnu++11
#
enable_testing()
#
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()
#
set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-sign-compare -lhts")
set(CMAKE_CXX_FLAGS_DEBUG "-g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

add_subdirectory(src)
#add_subdirectory(tests)

