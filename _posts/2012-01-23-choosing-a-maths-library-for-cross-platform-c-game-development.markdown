---
comments: true
date: 2012-01-23 15:35:28
layout: post
slug: choosing-a-maths-library-for-cross-platform-c-game-development
title: Choosing a maths library for cross-platform C++ game development
wordpress_id: 280
categories:
- Android
- Game Development
- Programming
tags:
- android
- C++
- cmake
- CML
- eigen
- GLM
- math
---

I have recently been looking for a C++ maths library for use in game development projects. There are plenty of posts on websites like [gamedev.stackexchange.com](http://gamedev.stackexchange.com/) with suggestions for libraries but few quantitative comparisons between them. I decided to take three of the most popular libraries and run some tests of my own. Below I describe the three libraries I've compared with their advantages and disadvantages and then show the results of some performance tests. It's important for me to have cross-platform compatibility, so each library selected is header only and has been tested on Mac, Linux, and Android using the NDK. The code and results are also available [on GitHub](https://github.com/mfoo/Math-Library-Test) for testing.

The libraries tested are:



	
  * [Eigen](http://eigen.tuxfamily.org/)

	
  * [GLM](http://glm.g-truc.net/)

	
  * [CML](http://cmldev.net/)


These choices are largely influenced by reading their websites and posts at the [Game Development StackExchange](http://gamedev.stackexchange.com/) site:



	
  * [Best C Math Library for Game Engine?](http://gamedev.stackexchange.com/questions/9924/best-c-math-library-for-game-engine)

	
  * [High Performance Math Library for Vector And Matrix Calculations](http://stackoverflow.com/questions/5935075/high-performance-math-library-for-vector-and-matrix-calculations)

	
  * [Complete Math Library for use in OpenGL ES 2.0 Game?](http://gamedev.stackexchange.com/questions/8234/complete-math-library-for-use-in-opengl-es-2-0-game)





# Requirements


I have several requirements for a math library, and when rating them (in no particular order) the [following things are important](http://gamedev.stackexchange.com/questions/21711/what-should-be-taken-into-consideration-when-choosing-a-math-library-for-games):



	
  * License

	
  * Portability

	
  * Quality and quantity of the documentation

	
  * Completeness (or how much can the library do without me having to write my own functions). For game development this includes:


	
    * Matrix operations

	
    * Vector operations

	
    * Complex number support

	
    * Quaternion operations


	
  * Extensibility and community contributions (linked directly to the above).

	
  * Ease of use (are the concepts simple to understand?)

	
  * Performance (are there SIMD optimisations?)




# Libraries Overview










## Configurable Math Library (CML)


This is a free library designed for games, graphics, and computational geometry applications. Listed [features](http://cmldev.net/?page_id=8) include:



	
  * Vector, matrix, and quarternion classes

	
  * Templated headers so can be used for arbitrary types

	
  * Arbitrary sized vectors and matrices (fixed or dynamically resizable)

	
  * Conversions between polar, cylindrical, spherical, and Cartesian coordinates

	
  * A large library of functions for the construction and manipulation of transforms in 2D and 3D


CML's [design notes](http://cmldev.net/?page_id=594) state that it is meant to be cross-platform and portable and therefore doesn't contain any platform specific optimisations, but it is possible to include them in the future if there is specific interest.

There's only one header file to include, `cml/cml.h`.

CML contains quite a nice way of creating an abstraction layer between existing math library objects and CML objects, allowing the use of other math library data types in CML library functions. See [here](http://cmldev.net/?p=424).

CML only has a few [examples](http://cmldev.net/?p=402). Hasn't been updated in a while. Contains useful functions for working with OpenGL or DirectX though, including replacements for things like gluLookAt().

CML is released under the [Boost Software License](http://cmldev.net/?p=430). This means that you're allowed to use and profit from the library as long as when distributing CML or any modifications you have made to CML, you keep the Boost Software License text in each file.


## Eigen


This library has by far the most detailed and descriptive tutorial section with many code samples. It is also worth noting that it seems to be the most frequently updated and the most up-to-date (at the time of writing the most recent release, 3.0.4, was 3.5 weeks ago).

Like GLM, it is very easy to pass the class objects directly to OpenGL, although in Eigen this is performed by an [unsupported OpenGL module](http://eigen.tuxfamily.org/dox-devel/unsupported/group__OpenGLSUpport__Module.html) which provides a few functions such as glTranslate and glRotate.

    
    // You need to add path_to_eigen/unsupported to your include path.
    #include <Eigen/OpenGLSupport>
    // ...
    Vector3f x, y;
    Matrix3f rot;
    glVertex(y + x * rot);
    
    Quaternion q;
    glRotate(q);


Provides a large number of array, matrix, and vector types with [many operators](http://eigen.tuxfamily.org/dox/QuickRefPage.html#QuickRef_Types).

Eigen is licensed under the LGPL which is quite restrictive and usually would exclude a library based on my requirements. However, Eigen has a large [Licensing FAQ](http://eigen.tuxfamily.org/index.php?title=Licensing_FAQ) which tries to answer any licensing questions people might have. Their main point is that as Eigen is a header only library, it does not count as a "Combined Work" under the LGPL it can be therefore used entirely under Section 3. This states that to use Eigen you must:



	
  * Give prominent notice with each copy of the object code that the Library is used in it and that the Library and its use are covered by this License. The Eigen FAQ states that 'the bottom of a README file or of a website would be prominent enough for us'.

	
  * Accompany the object code with a copy of the GNU GPL and this license document.


This assumes that you're using Eigen unmodified. If you are planning to make changes to Eigen and release it, then you must also make the modifications available under the LGPL.


## OpenGL Math Library (GLM)


GLM's main design to be very familiar to those who know GLSL as it uses classes and functions that use the GLSL naming conventions. It also seems to provide all of the functionality that I might need in the future.

Includes several code samples, more than CML. Many more located in the manual.

Includes features such as:



	
  * Math for splines

	
  * Math for colour spaces

	
  * Random numbers

	
  * Simplex noise generation

	
  * Conversion of Euler angles


There's only one header file to include, `glm/glm.hpp`. `glm/ext.hpp` can be included to add extended features (non GLSL features).

The design choice to follow GLSL conventions allows the library to be intuitive and easy to use, especially given that data alignment is compatible with gl functions. E.g:

    
    glm::vec4 v(0);
    glm::mat4 m(0);
    glVertex3fv(glm::value_ptr(v));
    glLoadMatrixfv(glm::value_ptr(m));


The manual pdf file link is broken but a slightly out of date version can be found [here](https://bitbucket.org/alfonse/gltut/src/6332c7f79903/glm-0.9.0.0/doc/glm-manual.pdf). I've sent the maintainer an email and hopefully this will be fixed shortly.

GLM is licensed under the [MIT License](http://en.wikipedia.org/wiki/MIT_License) (Expat License) which is very permissive and means GLM is a good candidate for any project requiring a math library.







To run the tests on Android you will need the [Android NDK](http://developer.android.com/sdk/ndk/index.html) and [android-cmake](http://code.google.com/p/android-cmake/). For Ant you will need to specify your Android SDK location in `android/local.properties`. I have included prebuilt libraries for armeabi and armeabi-v7a so if you don't want to compile them yourself you can skip straight to the ant commants.


# Results


So far I've tested matrix addition and multiplication. `src/Main.cpp` contains code that will generate two lists of 1 million 4x4 float matrices for each library, populate them with random float values, and then add each one from the first column to the second. It will then do the same for multiplication. It will repeat this step 10 times and print out how long it took for each library.

Results for each library vary greatly with architecture and optimisation level. I have tested standard GCC build on Mac OS X Lion as well as an SSE enabled build, and armeabi, armeabi-v7a and armeabi-v7a with NEON instructions for Android.

Note that all tests use the `-O2` GCC optimisation flag except the non-SSE laptop build which uses `-O0`.

Results for addition and multiplication are shown below. Note that the laptop I'm using is an i7 2.2ghz early 2011 MacBook Pro and the Android device is a Stock HTC Desire (2.2) with a 1 GHz Qualcomm QSD8250. All times are in milliseconds.

    
                            laptop  laptop (SSE)  armeabi  armeabi-v7a  armeabi-v7a with neon
    Eigen additions         8065    30            9944     2181         2145
    Eigen multiplications   22404   86            59460    5143         5113
    GLM additions           2375    76            10256    1506         1407
    GLM multiplications     7337    400           59008    2189         3108
    CML additions           12336   96            9587     2885         2996
    CML multiplications     21603   551           58399    5306         5280


The first column for the laptop doesn't have any compile-time optimisations and is included purely for interest. From the other results, Eigen seems to be the fastest for these operations, but GLM is the fastest on the HTC Desire with both ABIs. I am not sure why the NEON times are not much faster (and in some cases, slower) but will update this post when I learn more.

Despite GLM being faster on the mobile devices, I am more inclined to use Eigen due to its speed on the tested Intel CPU and its much better documentation and more active community.
