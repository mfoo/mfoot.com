---
comments: true
date: 2011-12-04 13:34:33
layout: post
slug: notes-on-android-graphics-and-animations
title: Notes on Android Graphics and Animations
wordpress_id: 244
categories:
- Android
- Game Development
---

This post contains my notes on the Youtube Video "[Learn about Android Graphics & Animations from Google's Android UI Toolkit Team](http://www.youtube.com/watch?v=duefsFTJXzc)" which shows a talk given by Romain Guy and Chet Haase from Google at the San Francisco Android User Group, Nov 20th 2010. Note that at the time of writing the video is already quite old, but much of it is still relevant and it's well worth a watch for anybody interested in working with graphics in Android. This post is partly for me to remember and refer back to, but hopefully others will find it useful. The video is embedded below and notes can be found after the break:

http://www.youtube.com/watch?v=duefsFTJXzc

So, summary of the video:



	
  * **Surface** is the name for the buffer that objects are rendered into, whether on-screen or off-screen.

	
  * 05:23: **PixelFlinger** is the equivalent of a JIT for the software implementation of OpenGL that's used in the Android emulator. This is pretty interesting, it generates assembly code at runtime based on what render operations you're performing.

	
  * 06:00: **View**s are the basic UI components, for instance each of the UI widgets inherits from View. **ViewGroup**s contain zero or more views and provide the base class for layouts and view containers, allowing you to modify the contents of the container and set properties for how the view children are displayed. **SurfaceView**s are special Views that allow you to place Surfaces at certain points in the screen. Android also provides classes such as the [GLSurfaceView](http://developer.android.com/reference/android/opengl/GLSurfaceView.html) which enables OpenGL to render into the Surface and the [VideoView](http://developer.android.com/reference/android/widget/VideoView.html) which allows videos to be rendered as well as providing video playback controls.

	
  * 06:40 Applications can render via a Canvas (which uses Google's Skia library) or via RenderScript (which currently makes OpenGL calls) or via OpenGL directly. Each rendering method eventually renders onto the specified kind of Surface.

	
  * 07:25 Android supports OpenGL ES 1.x and 2.0, and you can use either of these on a device that supports it, early devices and devices running versions before Android 2.2 will not support OpenGL ES 2.0 - see [here](https://secure.wikimedia.org/wikipedia/en/wiki/OpenGL_ES#Usage) for a list of devices that support each version. If you are using the Android Emulator then you will be using PixelFlinger which only supports OpenGL ES 1.x.

	
  * 08:15 Whenever a frame is drawn, **SurfaceFlinger** produces a composite from each visible View and renders them to a Surface (frame buffer) via either OpenGL or PixelFlinger depending on whether the software GL implementation is being used or not. Older devices used a 2D blitter called MDP.

	
  *  09:20 Android won't redraw a View unless it's both visible and flagged ('dirty') by calling it's [invalidate()](http://developer.android.com/reference/android/view/View.html#invalidate()) method. This invalidate call will propagate up to any ViewGroups in the view hierarchy to the **ViewRoot** which will lock the Surface of the window and then call draw() on all of it's children. When that's done the ViewRoot will unlock the canvas and swap the buffers. All windows on Android are double buffered.

	
  * 15:00 Demo from Chet about how to add animations and effects on Views. Animations can be used for subtle indications - for instance to indicate whether an image is selected or not it's nice to have an animation/transitioning effect rather than an abrupt change in the UI.

	
  * 16:56 Unlike Java2D, Android's Canvas is virtually stateless and uses **Paint**s to specify font size, text colour, colour, opacity, filtering, dithering, anti-aliasing etc. These objects contain a lot of state information and are therefore fairly heavy-weight. Making a new Paint object every frame is not a good idea as you will soon saturate the heap and cause Garbage Collection.

	
  * 18:15 **Shaders** specify how to draw horizontal or vertical spans of colours (for example different kinds of gradients), i.e. how to fill a shape. These are not the same as GLSL shaders. **ComposeShader** will blend two shaders together and fill the shape with that. **ColorFilter**s perform operations on each pixel they are applied to.

	
  * 21:00 **XferModes** or blending modes such as Porter-Duff (scientists who defined 12 equations that explain how transparency affects colours) and Darken, Lighten, Multiply and Screen allow modifying the colours of a Canvas. For instance, a BitmapShader and a LinearGradient can be combined with a ComposeShader to draw a bitmap that fades out.

	
  * 23:17 Demo from Chet showing some example code for using creating new Bitmaps that arise from applying Shaders on Bitmaps. Shaders are created and then the Paint object is told to use that shader. You can then simply draw a rectangle to the screen using the specified Paint and it will perform the shader operations you specified on that rectangle.

	
  * 25:00** Bitmap**s are either mutable or immutable and have the concept of resolution. Android allows you to specify different assets for different screen resolutions in the same package, and if you place a drawable in the medium resolution folder and are running on a phone which reports itself as high resolution, Android will use that information to scale the Bitmap automatically when it is displayed. Bitmaps can be large objects so they can be recycled without having to wait for the garbage collector. Supported Bitmap formats are **ALPHA_8 **(for alpha masks), **ARGB_4444** (this isn't recommended, uses 4 bits for each component and therefore doesn't look very good), **ARGB_8888** (recommended, 32 bit images, allows for transparency), and **RGB_565** (doesn't contain alpha channel, limited precision for colours, faster to draw, uses dithering). JPEG images do not contain transparency and pre-Gingerbread were loaded automatically with RGB_565. **This meant that jpg images automatically lost some quality pre-gingerbread**. Post Gingerbread all images are loaded by default as ARGB_8888 (and application memory usage limits are increased to compensate). When loading a Bitmap, make sure to specify the format that you want, otherwise it will be loaded as it's default and every time the Bitmap is drawn it will need to be converted. For instance, pre-Gingerbread the default bit depth for a Surface is 16 bits, so a 32 bit image will need to be converted before rendering which can be slow. You can control quality of this rendering by enabling/disabling dithering on the Paint object and the Drawable object that the Bitmap's being used by. Blending should be avoided with the alpha channel. If Android detects an image is completely opaque it can perform a faster rendering pass.

	
  * 30:00 Demo by Romain showing the effects of precision loss when rendering 32 bit images onto a 16 bit window (introduces artefacts, gradients look banded). Enabling dithering removes the banding somewhat and improves the gradient. Using a 32 bit window ARGB_8888 with no dithering looks fine while ARGB_4444 still looks bad (and hence isn't recommended anywhere).

	
  * 33:10 Slide showing performance comparisons for different bitmap image types on different surfaces. Drawing an ARGB_8888 surface is three times faster on a 32 bit surface than a 16 bit surface. ARGB_4444 is slightly faster than ARGB_8888 on a 16 bit surface. RGB_565 is 12 times faster than ARGB_8888 and 8 times faster than ARGB_4444 when rendering to a 16 bit surface (as it's essentially a memcpy on a 16 bit surface) but three times slower than ARGB_8888 and almost twice as slow as ARGB_4444 on a 32 bit surface.

	
  * 35:50 Demo of two different ways to copy a View into a Bitmap.

	
  * 37:30 Chet takes over, talks about Animations. Animations enable better user experiences and can help users use the application, especially on smaller screens. The current (as of the talk) SDK has an animation superclass which controls timing, repetition, interpolation and ending states. Nonlinear interpolation on animations such as view sliding is important as it looks much better than linear motion.

	
  * 40:30 Android allows for Transforming operations (translation, rotation, scale), Fading, Sequences (allowing you to choreograph multiple animations), Cross-fading, and Layout animations (ViewGroups can animate the display of their children via animations rather than displaying them instantly).

	
  * 41:40 Animations are about making things _look animated_ rather than actually _being_ animated. **This means that moving views via animations doesn't actually move the view. **If you move a button, you need to move the actual object afterwards to ensure that it will handle input events for the correct position. This is because the ViewGroup, when rendering it's children, detects first if an animation is playing on the child before rendering it. If it is, it renders the current state of the animation rather then the View's original settings.

	
  * 43:16 Fading is done by interpolating alpha values, which again is not the transparency of the View, but the transparency that the View will be drawn with (see previous point).

	
  * 45:30 Layout animations are based on staggering some template animation for each child of the view.

	
  * 46:50 With animations you can setDrawingCacheEnabled(True) to allow the animation to represent itself as a Bitmap which means that the animated object doesn't need to be re-rendered each frame. This is a large performance increase and is used in Android everywhere transparently. **As soon as a finger touches a view (for example a ScrollView) each of the Views in the ViewGroup are transformed into Bitmaps via setDrawingCacheEnabled which allows them to be scrolled quickly without having to re-render them.** When a Bitmap is off the screen it can be recycled and used by another View.

	
  * You can follow Romain Guy at @romainguy and curious-creature.org.

	
  * You can follow Chet Haase at @chethaase and graphics-geek.blogspot.com.




Slides of this talk can be found at [http://marakana.com/f/212](http://marakana.com/f/212).
