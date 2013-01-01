---
comments: true
date: 2011-12-14 15:03:15
layout: post
slug: cross-compiling-freetype-for-android-with-cmake
title: Cross-compiling FreeType for Android with CMake
wordpress_id: 255
categories:
- Android
- Programming
tags:
- android
- android-cmake
- cmake
- NDK
---

Freetype provides a Makefile with a configure utility that makes it fairly easy to cross-compile for other platforms, however there are a lot of examples on the web (as well as some [solutions](http://zarprime.blogspot.com/2006/08/how-to-cross-compile-freetype.html)) of people having difficulty in it.

My intention is to cross-compile FreeType using the Android NDK into a static library that can ultimately be used by [libRocket](http://librocket.com) for font rendering in HTML+CSS based UIs in OpenGL applications. I'm using the [CMake](http://cmake.org/) build system for cross-platform builds and the [android-cmake](http://android-cmake.googlecode.com) project to create a CMake toolchain for the NDK. This requires creating a CMakeLists.txt for FreeType as it doesn't currently provide one.

The CMakeLists.txt file included below is based on the FreeType-2.4.8 release and is a result of converting the Jamfile and Makefile to CMake. Its intention is simply to allow compilation with CMake. For standard build and installs I would recommend following the instructions in the docs/INSTALL file from the FreeType directory. **Note: **This file will try and build *all* of FreeType2's modules as defined in include/freetype/config/ftmodule.h. If you don't want to build some of these modules please edit that file and remove the macro calls.This is a rather brute force method that will compile all the files and modules that are in the default build settings for FreeType. I spent some time trying to add proper CMake style options for each module that also checks their dependencies but didn't get too far. You will need to remove files from compilation as needed.

    
    cmake_minimum_required(VERSION 2.6)
    
    project(FreeType2)
    
    # First, compiler definitions for building the library
    add_definitions(-DFT2_BUILD_LIBRARY)
    add_definitions("-DFT_CONFIG_MODULES_H=<ftmodule.h>")
    
    # Specify library include directories
    include_directories("${PROJECT_SOURCE_DIR}/builds/ansi")
    include_directories("${PROJECT_SOURCE_DIR}/include")
    include_directories("${PROJECT_SOURCE_DIR}/include/freetype")
    #include_directories("${PROJECT_SOURCE_DIR}/include/freetype/config")
    
    # For the auto-generated ftmodule.h file
    include_directories("${PROJECT_BINARY_DIR}/include")
    
    include_directories("${PROJECT_SOURCE_DIR}/objs")
    
    #file(GLOB BASE_SRCS "src/base/*.c")
    
    set(BASE_SRCS
        src/base/ftsystem.c
        src/base/ftdebug.c
        src/base/ftinit.c
        src/base/ftbbox.c
        src/base/ftbitmap.c
        src/base/ftcid.c
        src/base/ftadvanc.c
        src/base/ftcalc.c
        src/base/ftdbgmem.c
        src/base/ftgloadr.c
        src/base/ftobjs.c
        src/base/ftoutln.c
        src/base/ftrfork.c
        src/base/ftsnames.c
        src/base/ftstream.c
        src/base/fttrigon.c
        src/base/ftutil.c
        src/base/ftfstype.c
        src/base/ftgasp.c
        src/base/ftglyph.c
        src/base/ftgxval.c
        src/base/ftlcdfil.c
        src/base/ftmm.c
        src/base/ftotval.c
        src/base/ftpatent.c
        src/base/ftpfr.c
        src/base/ftstroke.c
        src/base/ftsynth.c
        src/base/fttype1.c
        src/base/ftwinfnt.c
        src/base/ftxf86.c
        src/truetype/truetype.c
        src/type1/type1.c
        src/cff/cff.c
        src/cid/type1cid.c
        src/pfr/pfr.c
        src/type42/type42.c
        src/winfonts/winfnt.c
        src/pcf/pcf.c
        src/bdf/bdf.c
        src/sfnt/sfnt.c
        src/autofit/autofit.c
        src/pshinter/pshinter.c
        src/raster/raster.c
        src/smooth/smooth.c
        src/cache/ftcache.c
        src/gzip/ftgzip.c
        src/lzw/ftlzw.c
        src/bzip2/ftbzip2.c
        src/psaux/psaux.c
        src/psnames/psmodule.c)
    
    include_directories("src/truetype")
    include_directories("src/sfnt")
    include_directories("src/autofit")
    include_directories("src/smooth")
    include_directories("src/raster")
    include_directories("src/psaux")
    include_directories("src/psnames")
    
    add_library(freetype SHARED ${BASE_SRCS})
    
    set(FREETYPE_LIBRARY freetype CACHE STRING "The FreeType library name")
    set(FREETYPE_FOUND TRUE CACHE BOOL "Whether freetype has been found or not")
    set(FREETYPE_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include/
        ${PROJECT_SOURCE_DIR}/include/freetype CACHE STRING "FreeType include
        directories")
    
        set(FREETYPE_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/include/
        ${PROJECT_SOURCE_DIR}/include/freetype CACHE STRING "FreeType include
        directories")


The file sets the variables that FindFreeType.cmake sets, so any dependent sub-projects should use this first. Just remember to use:

    
    # Build FreeType
    add_subdirectory(lib/freetype-2.4.8)
    include_directories(lib/freetype-2.4.8/include)


... before building the dependent projects.
