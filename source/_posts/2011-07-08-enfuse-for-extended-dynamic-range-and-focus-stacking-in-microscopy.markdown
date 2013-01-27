---
comments: true
date: 2011-07-08 03:09:53
layout: post
slug: enfuse-for-extended-dynamic-range-and-focus-stacking-in-microscopy
title: Enfuse for Extended Dynamic Range and Focus Stacking in Microscopy
wordpress_id: 116
categories:
- Photography
tags:
- bee
- enfuse
- exposure fusion
- focus stack
- microscope
---

[caption id="attachment_146" align="aligncenter" width="656" caption="The right eye and antennae of a bee produced from a stack of different photographs combined with Enfuse."][![Bee Closeup](/images/bee_closeup.png)](/images/bee_closeup.png)[/caption]

[Enfuse](http://enblend.sourceforge.net/) is a piece of open source software that is designed for combining multiple images containing different exposures of the same scene into a single image that is well-exposed as possible. This process is called exposure fusion, not to be confused with the popular high dynamic range (HDR) techniques. HDR techniques build a single high dynamic range source image and then apply a tone mapping operator to compress it into a range that a monitor can display or a printer can print. _Enfuse_ skips this step and tries to combine pixels from each source image directly into an output image by assigning each pixel a weighting based on contrast, exposure and saturation calculations in the pixel's local area. Pixels from each coordinate in the image stack are then combined based on their weightings.

This method of combining pixels means that not only is _Enfuse_ good at combining multiple exposures, but it is good at combining focus stacks --- images that are focussed on different parts of the subject. By weighting the decision heavily on local neighborhood contrast rather than saturation and exposure and telling _Enfuse_ to only use the pixel with the highest weighting from the stack (rather than combining pixels in the stack based on weighting) we can perform focus stacking.

I recently tried _Enfuse_ on images captured directly from a microscope and have shown that it's very useful when the subject contains both light and dark (and matte and glossy) areas where it's difficult to illuminate correctly with a single lighting setup. The images below show six different exposures of a piece of mould growing on a coffee cup that was in the PhD lab at the time. There's a crack in the sample and the light from underneath shines through, making it difficult to see the image detail in that area without underexposing the top.


[![](/images/1031_small.png)](/images/1031_small.png)[![](/images/1030_small.png)](/images/1030_small.png)[![](/images/1029_small.png)](/images/1029_small.png)




[![](/images/1028_small.png)](/images/1028_small.png)[![](/images/1027_small.png)](/images/1027_small.png)[![](/images/1026_small.png)](/images/1026_small.png)


When these are combined in _Enfuse_ we get an output that looks like the one below with no over- or under-exposed areas.

[caption id="attachment_131" align="aligncenter" width="400" caption="Exposure fused composite"][![Exposure fused composite](/images/hdr181031_small.jpg)](/images/hdr181031_small.jpg)[/caption]

I wrote a Python wrapper for the microscope-mounted camera to capture photos and another Python wrapper for the stage's RS-232 instruction set and set up an automated process to take six different exposures at a time and then move the stage in the Z axis and repeat. After combining each of the six images into an exposure-fused output and then combining those images in a focus stack we can produce something like this:

[caption id="attachment_132" align="aligncenter" width="400" caption="Output of using Enfuse to focus stack a stack of exposure-fused images. Note that the lines in the image aren't produced by Enfuse, there were some problems with the data capture and some parts of the image were lost."][![Stacked Mould Images](/images/stacked18.png)](/images/stacked18.png)[/caption]

Now we've shown that focus stacking works nicely on exposure fused images from the microscope, we can expand the program to move the stage around and build up a mosaic as shown below.

[caption id="attachment_136" align="aligncenter" width="700" caption="A composite of 9 different exposure fused focus stacks."][![Mould Mosaic](/images/mouldPanoIM.jpg)](/images/mouldPanoIM.jpg)[/caption]

With a different subject with a much greater physical depth (more Z-steps) we can produce something that looks like the bee below. This image is a composite of 3600 source images that were automatically combined with _Enfuse_ and then were stitched together into the mosaic manually. The output is something that would never be visible to the human eye due to physical limitations of the lens, and the final composite is around 90 megapixels. The image here is an incredibly reduced version of it. A full resolution version can be found [here (warning: 10 megabytes jpg file, 7680x11728px, may crash your browser)](/images/bee.jpg).

[caption id="attachment_145" align="aligncenter" width="397" caption="A composite of 3600 images fused with Enfuse and manually stitched."][![Bee Mosaic](/images/bee1.png)](/images/bee1.png)[/caption]
