---
layout: post
title: "Light Painting with the Raspberry Pi"
date: 2013-01-30 22:34
comments: true
categories:
- Photography
- Raspberry Pi
- Electronics
tags:
- raspberry pi
- light painting
- python
- led stick
- arduino
---

![](/images/DSC07335.png)

I recently came across some images from Flickr user
[TxPilot](http://www.flickr.com/photos/txross/sets/72157622234691540/), who has
created a light painting wand controlled by an Arduino and has used it to
create some stunning photos such as the one below.

![](http://24.media.tumblr.com/tumblr_lnz4766sYj1qbg80vo1_500.jpg)

This seemed like a brilliant idea, so I set about making my own. I used a
Raspberry Pi rather than an Arduino as prototyping was faster and easier, and
it allows me to do a bunch of things that the Arduino would not. The basic idea
is that you provide an image file and the light stick will show that image file
column by column. If you mount your camera on a tripod and hold the shutter
open while moving the light stick along, it reproduces the image in a light
trail. The images below show an example input file, and the kind of thing you
can produce with it:


Source image:

![Gradient fill](/images/lightstick-gradient.png 100 32)

Results:

![Light stick example](/images/DSC07346.png)

![Light stick example](/images/DSC07348.png)

The Pi is using a cheap USB wifi dongle and runs a DHCP a server. When it
boots, it sets up an ad-hoc wireless network and boots a small Python web
application that provides a control interface to the light wand. This allows
anybody with a computer or smartphone to connect to the network, upload an
image to the Pi, and control the playback of that image on the light stick. The
whole thing runs from a portable battery that's designed for charging your
devices while you're away from a plug for a long time.

#Technical Information

![A screenshot of the web app from an Android device](/images/rpi-light-painting-screenshot.png 232 412)

The Raspberry Pi runs a small Python application written using the
[web.py](http://webpy.org/) framework. It uses [Twitter Bootstrap](http://twitter.github.com/bootstrap/)
for style, layout, and responsive design, and the
[RPi.GPIO](http://pypi.python.org/pypi/RPi.GPIO/) Python module for
communicating with the lights. The source code for the Python application can
be found [here](https://github.com/mfoo/rpi-light-painting). The lights
themselves consist of a flexible strip of 32 individually addressable RGB
LEDs.

The Pi communicates using the GPIO pins via SPI to the LED strip, but the RPi
outputs 3.3v on these pins. I didn't have any transistors to hand so I used couple
of NOT gates in a 7404 hex inverter chip to bump the logic level to 5v. I don't
need to read from the strip, the 3.3v is higher than the switching voltage for
the hex inverter, and the GPIO frequency is pretty low so the 7404 was fine for
the job.

You can see an image of the web interface on the right. It is very simple, and
consists of a few buttons that control the state of the application (these are
shown and hidden at the appropriate times to stop you pressing the wrong
thing), a file upload form and a gallery of existing uploaded images. Clicking
an image in the gallery will start playing it.

#Parts List and Cost Breakdown

The list of parts can be found below (shipping not included):

* [Raspberry Pi](http://uk.farnell.com/raspberry-pi) - £26.78
* [Cute RPi case](http://pibow.com/) - £12.95 (optional!)
* [TP-Link Wireless USB
  Dongle](http://www.amazon.co.uk/TP-Link-TL-WN821N-Wireless-USB-Adapter/dp/B00194XKXA/ref=sr_1_8?ie=UTF8&qid=1359583224&sr=8-8) - £9.87
* [USB Battery Charger Power
  Supply](http://www.amazon.co.uk/Upgraded-External-Flashlight-Lightning-Provided/dp/B0067UPRQ4/ref=sr_1_3?ie=UTF8&qid=1359583352&sr=8-3) - £23.99.
* [Weatherproof Digital RGB LED strip](http://adafruit.com/products/306) - £19

I also spent about £6 at a local DIY store for a 1m section of 3/4" clear PVC
pipe and 1m of stiff plastic pipe thing that stops it from bending. I used
these to construct a clear backbone+shell for the flexible light strip that is
entirely waterproof. Total cost comes to about £100 incl. postage. Add a
little more if you need prototyping boards and components. The
[Raspberry Pi GPIO breakout kit](http://www.hobbytronics.co.uk/raspberry-pi-gpio-breakout)
is a good choice for connecting the Pi to a breadboard.

Note: The Anker USB power supply is a good choice because it has both a 1A and
a 2A output, both rated at 5v. They market this because it can charge your
devices faster, but at ~20 mA per LED for a 1m 32-LED strip + ~500mA for the
RPi + the wifi dongle, the 2A connection is necessary for this build.

![](/images/raspi-lightstick-1.jpg)

![](/images/raspi-lightstick-2.jpg)
