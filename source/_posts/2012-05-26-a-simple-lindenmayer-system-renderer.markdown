---
comments: true
date: 2012-05-26 08:23:28
layout: post
slug: a-simple-lindenmayer-system-renderer
title: A Simple Lindenmayer System Renderer
wordpress_id: 333
---

I spent some time last week reading about [Lindenmayer Systems](http://en.wikipedia.org/wiki/L-system) (or L-Systems), then wrote a simple interactive L-System renderer in CoffeeScript that you can try out [on GitHub](http://mfoo.github.com/L-System-Renderer/).

[![](/images/Screen-Shot-2012-05-26-at-08.02.06-1024x919.png)](/images/Screen-Shot-2012-05-26-at-08.02.06.png)



L-Systems are a way of describing an image in terms of an axiom, a set of generation functions, and a set of rendering functions. They are represented as a string, and for each 'generation', the generation functions are applied to all symbols in that string to produce a new string. For example, given the axiom (starting truth) 'A' and the generation functions 'A -> BA', and 'B -> AB', generation one will be 'BA', then generation two will be 'ABBA'.

We can then define a set of rendering functions that define what to draw when we reach each symbol such as 'move forward 10 units' and 'turn left 20 degrees'. With each generation the system gets bigger and bigger, and it's fantastic to play with.

For instance, the code below describes how to draw a [Sierpinski Triangle ](http://en.wikipedia.org/wiki/Sierpinski_triangle)using 'turtle graphics' and a simple [stack](http://en.wikipedia.org/wiki/Stack_%28abstract_data_type%29):

    
        'Sierpinski Triangle':
            axiom: 'A'
            rules:
                'A': 'B-A-B'
                'B': 'A+B+A'
            renderFunctions:
                'A': (stack) ->
                    turtle = stack.peek()
                    turtle.forward 10
                'B': (stack) ->
                    turtle = stack.peek()
                    turtle.forward 10
                '-': (stack) ->
                    turtle = stack.peek()
                    turtle.left 60
                '+': (stack) ->
                    turtle = stack.peek()
                    turtle.right 60


Which, when rendered for five generations looks like this:

[![](/images/sierpinskitriangle-300x166.png)](/images/sierpinskitriangle.png)The source code for the render is available [here](https://github.com/mfoo/L-System-Renderer).


