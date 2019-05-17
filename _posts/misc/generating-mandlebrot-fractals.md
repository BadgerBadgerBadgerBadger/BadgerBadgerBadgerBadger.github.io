---
layout: post
title: Generating Mandelbrot Sets with Processing
excerpt: "My explorations with the Mandelbrot Set, visualising it, playing around with it, etc."
modified: 2019-05-17T14:17:25-04:00
categories: misc
tags: [kotlin, processing, generative-art]
comments: true
share: true
---

This is going to be a post about some exploratory stuff that I did once I learned of just how cool the Mandelbrot Set is. I had known of the set for a long time, seen trippy visualisations (as someone pointed out, youtube is full of them), but this was the first time I felt like attempting it myself.

1. [The Mandelbrot Set](#the-mandelbrot-set)
1. [First Attempt](#first-attempt)
1. [Second Attempt](#second-attempt)

## The Mandelbrot Set

I'm not going to talk about the Mandelbrot Set myself. There are much better and more interesting explanations out there. I'll link you guys to those and you can come back once you've had your fill. These are some of the things that inspired me and some of the resources that I looked up afterwards. I'll put them in the same order I encountered them.

1. [What's so special about the Mandelbrot Set? - Numberphile](https://www.youtube.com/watch?v=FFftmWSzgmk) - A beautiful video with lovely visualizations that made me want to start playing around with processing and generating my own visualisations.
1. [The Mandelbrot Set - Numberphile](https://www.youtube.com/watch?v=NGMRB4O922I&t=16s) - More about the Mandelbrot Set.
1. [Mandelbrot Set from Wolfram Mathworld](http://mathworld.wolfram.com/MandelbrotSet.html) - I tried to grok this and then promptly gave up on it.
1. [Mandelbrot Set on Vsauce](https://www.youtube.com/watch?v=MwjsO6aniig) - A very entertaining and education video by the beloved Michael Stevens.
1. [Understanding Julia and Mandelbrot Sets by Karl Sims](https://www.karlsims.com/julia.html) - This one helped me a _lot_.

## First Attempt

> I am going to assume you've seen enough of the above videos/articles that I won't have to repeat the math behind it.

I first started with [p5js](https://p5js.org) on https://editor.p5js. Here's my [Sketch](https://editor.p5js.org/scionofbytes/full/UfCfqKVrY). As you can no doubt tell from the commented out bits and the debugging functions, this is a very rough attempt at getting something going. With a window size of 450x200, the rendering happens pretty fast, but nowhere close to real-time (which, for me, needs to be at least 30 fps).

I suspect this is because I've made several mistakes here. I'm iterating way too high (200!), actually waiting for the numbers to hit Infinity (rather than stopping as soon as they go over 2), using a complex package with a whole lot of weird code rather than rolling a very minimal solution since I need only complex multiplication and addition, which, with some algebra, could be done much more efficiently rather than allocating an entirely new complex number type. But, anyway, I wasn't thinking of all that, I was going for the fastest implementation rather than the fastest program.

The result turned out to be this thing that works, but isn't particularly performant.

I also have a version of this with cleaner code on https://openprocessing.org ([this one](https://www.openprocessing.org/sketch/707203)), but I stressed it out too much in terms of resolution and openprocessing is dumb in that it just runs your sketch by default. So as soon as that page opens I can no longer do anything because my stupid code hangs the page. Lovely.

This is how it looks, by the way:
![p5 editor result](https://i.imgur.com/rKzuPXj.png)

Not too shabby. The basic shape is there, it's a bit rough around the edges because of the low resolution.

## Second Attempt

I decided I needed more juice so instead of doing this in Javascript, I'd do this in [Processing 3](https://processing.org). I'd have the JVM and the native performance that would entail. But I find the editor to be incredibly clunky and I'm far too used to my Jetbrains IDEs. So I fire up IDEA, got the Processing library, set everything up for Kotlin (yes, there's no way I'm programming in Java, eat my shorts), and got cracking at it.

The same visualisations, using Kotlin and on the JVM, with the image being saved to a png instead of being displayed on screen. I planned to go for much higher resolutions, so I thought starting with saving the images to disk directly might be the way to go.

Anyway, so this is what that ended up looking like:

![kotlin version low res](https://i.imgur.com/QyoLM9U.png)
This is the first version, lower resolution.

And then there's a slighlty higher resolution one.

![kotlin version higher res](https://i.imgur.com/OxFtnsN.png)

These took way too long to generate, the first one clocking in at 2251ms, the second at 7869ms! Hecking no way that's anywhere close to real-time.

I'm still making mistakes here. Still using a full package to do simple complex math (it's not as contradictory as it sounds especially if you call it _imaginary_ math instead of _complex_ math). I did stop waiting for the computed value to grow to Infinity. This time I'm stopping as soon as it grows above 2. But no performance gain over the JS version at all. I'm probably doing somethind really dumb here. But before I go into that, let me tell you what I tried to do.

I tried to generate a 12000x10000 image. That's 120 million pixels. A 120 mega-pixel image. I tried to generate that on my laptop. And I made a royal mess of it, too. Not only did it take over an hour to do, I forgot to save it, so just to show you guys that image I'll have to let my laptop run hot for an hour and it won't even be that great an image. You'll get to see it anyway.

But for that we'll go step by step. I'm going to use the same crappy code through all the steps and only change a few things here and there.

First a 1200x1000 that took 38,785ms:

![kotlin version 1200x1000](https://i.imgur.com/nI51GLq.png)

This is pretty nice looking, actually, but let's see if we can do something with colors!
