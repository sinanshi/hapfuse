include_directories(${CMAKE_SOURCE_DIR}/src)
file(GLOB_RECURSE hapfuse_SRC *.cpp)

#add_library(hapfuse_ SHARED ${hapfuse_SRC})
add_executable(hapfuse ${hapfuse_SRC})
target_link_libraries(hapfuse ${Boost_LIBRARIES})
target_link_libraries(hapfuse ${Threads_LIBRARIES})
target_link_libraries(hapfuse ${ZLIB_LIBRARIES})
#target_link_libraries(hapfuse hapfuse_)
#target_link_libraries(libhapfuse ${Boost_LIBRARIES})
#target_link_libraries(libhapfuse ${Threads_LIBRARIES})
#target_link_libraries(libhapfuse ${ZLIB_LIBRARIES})


if(THREADS_HAVE_PTHREAD_ARG)
  target_compile_options(hapfuse PUBLIC "-pthread")
endif()

if(CMAKE_THREAD_LIBS_INIT)
  target_link_libraries(hapfuse "${CMAKE_THREAD_LIBS_INIT}")
endif()
