---
comments: true
date: 2011-12-14 14:52:32
layout: post
slug: building-boost-1-47-for-android-using-cmake-and-the-ndk
title: Building Boost 1.47 for Android using CMake and the NDK
wordpress_id: 256
categories:
- Android
- Programming
tags:
- android
- android-cmake
- boost
- cmake
- NDK
---

The [Boost library](http://www.boost.org/) is incredibly useful in cross-platform C++ software development. Building Boost for Android can be a bit troublesome and several patches need to be applied to the code. Mystic Games provides a project on GitHub called [Boost for Android](https://github.com/MysticTreeGames/Boost-for-Android) which at the time of writing worked with official NDK r5c and Boost version 1.45. See after the break for more information.

Version 1.47 adds some useful modules such as Boost::Random and Boost::Asio, but the patches from 1.45 don't apply. Luckily klayge.org has applied the patches manually (there aren't many).


## Patching Boost:





	
  * The patched Boost 1.47 files [here](http://www.klayge.org/2011/11/02/compile-boost-1-47-with-android-ndk-r6/).

	
  * The Boost 1.47 source [here](http://www.boost.org/users/history/version_1_47_0.html).

	
  * [android-cmake](android-cmake.googlecode.com) installed and configured to use your NDK toolchain.


First, merge (don't replace!) the _boost_ and _libs_ folders from the download into the boost directory. You now have a patched version of Boost. Paste the contents of the code below into a file called CMakeLists.txt inside the _boost_ directory. This is a modified version of android-cmake's Boost installation script that allows you to either build and install Boost to your NDK directory or build it as a dependency for a project (this has the advantage of being sure you're using the same version of Boost on each machine you build on and for each architecture, as well as meaning you don't need to manually install Boost on each development machine, but Boost is a very large dependency to add and it is a lot of files to place into a repository).

    
    cmake_minimum_required(VERSION 2.8)
    
    project(android-boost)
    
    #find patched boost directory
    set(BOOST_ROOT ${PROJECT_SOURCE_DIR} CACHE PATH  "Boost 1.47.0 patched for android")
    
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -DNO_BZIP2" )
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DNO_BZIP2")
    
    include_directories(${PROJECT_SOURCE_DIR})
    
    # Build each of the Boost libraries that have compiled components
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/system/src/*.cpp)
    
    add_library( boost_system ${lib_srcs})
    
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/filesystem/v2/src/*.cpp)
    
    add_library( boost_filesystem ${lib_srcs})
    
    set(lib_dir ${PROJECT_SOURCE_DIR}/libs/iostreams/src)
    set(lib_srcs ${lib_dir}/file_descriptor.cpp   ${lib_dir}/gzip.cpp   ${lib_dir}/mapped_file.cpp   ${lib_dir}/zlib.cpp)
    add_library( boost_iostreams ${lib_srcs})
    
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/program_options/src/*.cpp)
    
    add_library( boost_program_options ${lib_srcs})
    
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/regex/src/*.cpp)
    
    add_library( boost_regex ${lib_srcs})
    
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/signals/src/*.cpp)
    
    add_library( boost_signals ${lib_srcs})
    
    file(GLOB lib_srcs ${PROJECT_SOURCE_DIR}/libs/thread/src/pthread/*.cpp)
    
    add_library( boost_thread ${lib_srcs})
    
    set(BOOST_ROOT ${PROJECT_SOURCE_DIR} CACHE PATH "Path to the Boost root directory")
    set(BOOST_INCLUDEDIR ${BOOST_ROOT} CACHE PATH "Boost include directory")
    set(Boost_INCLUDE_DIRS ${BOOST_ROOT} CACHE PATH "Boost header locations")
    set(Boost_LIBRARIES boost_filesystem boost_system boost_program_options
    boost_iostreams boost_regex boost_signals boost_thread  CACHE STRING "khkh")
    
    configure_file(${PROJECT_SOURCE_DIR}/BoostConfig.cmake.in
    ${PROJECT_BINARY_DIR}/BoostConfig.cmake @ONLY)
    
    install(DIRECTORY ${BOOST_ROOT}/boost DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
    install(TARGETS boost_system DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_filesystem DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_program_options DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_iostreams DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_regex DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_signals DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
    install(TARGETS boost_thread DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)


Optional: You can then also add Boost 1.47's version number to the known versions in FindBoost.cmake (Boost 1.47 is newer than CMake 2.8 that I am using):

    
      set(_Boost_KNOWN_VERSIONS ${Boost_ADDITIONAL_VERSIONS}
        <strong>"1.47.0" "1.47"</strong> "1.46.0" "1.46" "1.45.0" "1.45" "1.44.0" "1.44" "1.43.0" "1.43" "1.42.0" "1.42"




## Option: Building Boost for the NDK


You can then either open a terminal and build Boost using android-cmake and install to the NDK directory:

    
    $ cd boost_1_47/
    $ mkdir build
    $ cd build/
    $ android-cmake ..
    $ make
    $ sudo make install


Boost should now be able to be found by CMake with the standard find_package command:

    
    find_package(Boost 1.47 COMPONENTS filesystem system thread REQUIRED)




## Alternative: Adding Boost as a sub-project in CMake


If you move the boost directory into your third party library directory you can then build boost and be sure you have the correct version all of the time by adding this to your project's CMakeLists.txt file:

    
    # Build boost 1.47
    add_subdirectory(lib/boost_1_47)


Just remember to link against ${Boost_LIBRARIES} and include ${Boost_INCLUDE_DIRS} as you usually would.
