---
comments: true
date: 2011-07-10 05:35:52
layout: post
slug: using-doxygen-and-google-c-testing-framework-with-cmake
title: Using Doxygen and Google C++ Testing Framework with CMake
wordpress_id: 169
categories:
- Programming
tags:
- C++
- cmake
- ctest
- doxygen
- googletest
- gtest
---

**What is CMake?**

[CMake](http://www.cmake.org) is a build system that makes it incredibly easy to distribute and compile source code packages on multiple platforms. It will automatically scan dependencies, searching in the correct (for standard installs) places for each platform and warning if they are not met. Once it has calculated what's necessary to build the project it can generate build scripts and project files for several popular IDEs including Eclipse, XCode, Unix Makefiles, and Visual Studio ([read their about page to learn more](http://www.cmake.org/cmake/project/about.html)).



**CMake with Doxygen**

As well as building your source, CMake can also run [Doxygen](http://www.stack.nl/~dimitri/doxygen/) to generate documentation after a build. Here's a sample that I use based on the example [here](http://majewsky.wordpress.com/2010/08/14/tip-of-the-day-cmake-and-doxygen/):

[code]
################################################################################
# Documentation Generation
#
# Build documentation using Doxygen (www.doxygen.org)
# Builds the docs in the docs/ directory (HTML and LaTeX formats for a .pdf)
################################################################################
find_package(Doxygen)

if(DOXYGEN_FOUND)
    configure_file(${PROJECT_SOURCE_DIR}/doc/Doxyfile.in ${PROJECT_SOURCE_DIR}/doc/Doxyfile @ONLY)
    add_custom_target(doc ALL ${DOXYGEN_EXECUTABLE} ${PROJECT_SOURCE_DIR}/doc/Doxyfile  COMMENT "Generating API documentation with Doxygen" VERBATIM)
endif(DOXYGEN_FOUND)
[/code]

This will take a file called Doxyfile.in (rename your Doxyfile if you're just getting started) and substitute any values that you've specified into the file. This means that by changing a variable in the CMakeLists.txt you can have that change show up in your documentation. Inside the Doxyfile.in file you can specify things like this:

[code]
PROJECT_NAME = "@PROJECT_NAME@"
PROJECT_NUMBER = "@PROJECT_VERSION_MAJOR@.@PROJECT_VERSION_MINOR@"
PROJECT_LOGO = @CMAKE_CURRENT_SOURCE_DIR@/assets/project_logo.png
OUTPUT_DIRECTORY = @CMAKE_CURRENT_BINARY_DIR@/doc
INPUT                  = @CMAKE_CURRENT_SOURCE_DIR@/src @CMAKE_CURRENT_SOURCE_DIR@/test/src @CMAKE_CURRENT_SOURCE_DIR@/src/Core.h.in
EXAMPLE_PATH = @CMAKE_CURRENT_SOURCE_DIR@/examples
IMAGE_PATH = @CMAKE_CURRENT_SOURCE_DIR@/assets
[/code]

**CMake and CTest with Google C++ Testing Framework**

CMake makes it very easy to add other projects that use CMake to the build as a dependency. [Google's C++ Testing Framework](http://code.google.com/p/googletest/) provides a CMakeLists.txt, so all you need to do in order to add gtest to your project is point your CMakeLists.txt to the gtest folder. Mine's in the 'lib' folder:

[code]
################################################################################
# Testing
#
# Build GTest (See http://code.google.com/p/googletest/)
# Run different GTest test programs.
################################################################################

enable_testing(true)

# Build GTest
add_subdirectory(lib/gtest-1.6.0)

include_directories(${gtest_SOURCE_DIR} ${gtest_SOURCE_DIR}/include)

# Compile the test sources.
add_executable(runTests test/src/test.cpp)

# Link the test sources to the gtest libraries.
target_link_libraries(runTests ${LIB} gtest gtest_main)

# Add that executable to the CTests testing framework.
add_test(
    FixedSizeArrayTests runTests
)
[/code]

The only problem here is that while in GTest you don't need to enumerate each file, using CMake's CTest facility you will do. This could be fixed by defining a simple macro that will add all the tests in a folder. [This is a good starting point.](https://code.ros.org/svn/opencv/trunk/opencv/samples/cpp/CMakeLists.txt) The above code will enable the 'test' build target in the Makefile.

In addition, gtest provides the option to output it's test results to JUnit-compliant XML files. In order to do this you need to have a minimum of CMake version 2.8 (so set cmake_minimum_required(VERSION 2.8)) and then you can set the CTest environment variable "xml" like this:

[code]
set_tests_properties(FixedSizeArrayTests PROPERTIES ENVIRONMENT "GTEST_OUTPUT=xml")
[/code]

This is especially useful if you're using a build system such as [Jenkins](http://www.jenkins-ci.org) (if so, be sure to look at the [CMakebuilder plugin](https://wiki.jenkins-ci.org/display/JENKINS/cmakebuilder+Plugin)) as you can set Jenkins to perform testing and then display JUnit test results with each build.
