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

## Intro

Around a few weeks ago, somewhere in late April of 2019, I became fascinated with the Mandelbrot Set. I came across an interesting video by [Ben Sparks](https://www.bensparks.co.uk/) on [Numberphile](https://www.youtube.com/watch?v=FFftmWSzgmk) and he spoke of the Mandelbrot Set. He starts off with the idea of stable iterations, first on a one-dimensional number line and then on a two dimensional complex plane, and demonstrates how, on iterating on the square of a given number, the result can either blow up and tend towards infinity or remain stable and tend towards 0. Or at least that's the idea I got. Someone more mathy correct me if I'm wrong.

The visualisations in the video were quite nicely done and I got an itch to try them myself. My best friend has been studying Daniel Shiffman's [Code! Programming with p5.js for Beginners](https://www.youtube.com/watch?v=HerCR8bw_GE&list=PLRqwX-V7Uu6Zy51Q-x9tMWIv9cueOFTFA&index=1) and that reminded me of my own early years in programming when I kept myself motivated by building trippy animations and other cool stuff you can do with a library like p5js (though back then only [Processing](https://processing.org) existed).

So I thought, why not try my hand at generating the Mandelbrot Set?

## Complex Numbers

But first we need to talk about Complex Numbers. There are [a ton](https://www.youtube.com/watch?v=gHUHZXjpwOE) [of videos](https://www.youtube.com/watch?v=sZrOxm5Gszk) [on youtube](https://www.youtube.com/watch?v=SP-YJe7Vldo) about Complex Numbers. Going into the details of why the exist, myself, is not that great an idea. Let's just say that for our purposes we can think of complex numbers existing on a line perpindicular to our regular number line. If you take a piece of graph paper and mark a line from left to right, and then another line that crosses that line at 90 degress, from top to bottom, you can imagine that the horizontal line represents the real number line, and the vertical line represents the complex number line. And thus we have a complex plane to work with.

I highly suggest watching the videos I've linked to to really grasp the ideas behind Complex Numbers and how they represent a plane, but for our purposes let's just say we have a grid and two numbers to work with, one representing a horizontal position on the grid, and one representing a vertical. And these give us a point.

## The Mandelbrot Set

The Mandelbrot set starts with the idea of iteration. Say you have a number **z**. This is a complex number of two parts, but for now let's imagine it's a simple number.

Now let's square **z**. Well, that's simple enough. Squaring **z** gives us **z<sup>2</sup>**. Let's take another number **c** and add it to our result to get **z<sup>2</sup> + c**. Let this be our new z.

So we have the operation **z' = z<sup>2</sup> + c**
And then we can do this again but now instead of our original **z** we are using our new value **z'**. This we way keep feeding the result of the previous operation into the next one. We keep iterating like this untill we have a reson to stop. We can start with a value of **z = 0**, but it's the **c** that's the important number, here.

The Mandelbrot Set, as I understand it, is the set of all numbers in the Complex Plane that, when put under this iteration, does not blow up. The **c** in our equation is the number we're testing The not-blowing-up part is important, we'll come back to that.

So, when speaking of the Mandelbrot set what we do is, for each point in our grid, we test to see if, when put under iteration via the **z' = z<sup>2</sup> + c** operation, where **c = the point**, if the value blows up, then the point is not part of the Mandelbrot Set. And if it does not, then it is.

Honestly, this is all very mathy and I am going to stop with the Math now. I highly recommend you watch the above videos for a solid understanding of Complex Numbers and the Mandelbrot Set. From here on out I will be talking mostly code. 

Let's say we have a grid of pixels, and each pixel can be represented by coordinates. Let's say also that we have a function (the programming kind, not the math kind) `testMandelbrot` which takes a coordinate `c` and tells us if `c` is in the Mandelbrot Set or not.

And so we proceed.

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

Just so that we are aware, 200 is the number of iterations at which point I stop checking if the value blew up.

First a 1200x1000 that took 38,785ms:

![kotlin version 1200x1000](https://i.imgur.com/nI51GLq.png)

This is pretty nice looking, actually, but let's see if we can do something prettier.

![kotlin version 1200x1000 with extra greys](https://i.imgur.com/fHBJM8q.png)

For each of the points _not_ inside the Mandelbrot set (so those for which the equation blows up above 2), I'm taking the number of iterations it took before it blew up and mapping that from 0 - 200 to 0 - 255 (greyscale color range).

Now we can not only see the scattered Mandelbrot islands, we can also see the possible connections. The connections themselves are probably too small to show up in the visualisation, but once we show the borders in grey, they show up pretty nicely.

Let's try something in another color.

![kotlin version 1200x1000 with red borders](https://i.imgur.com/d0Mt31G.png)

So for this I decided to color the inside of the Set fully black, and let the outside be mapped from 0 (full black) to #FF0000(full red), giving us a vivid red border.

 > Side note: I wonder why it's so transparent. I think I messed up an alpha channel somewhere.
 
 Okay, let's do something even more colorful.
 
![kotlin version 1200x1000 with multi-color borders](https://i.imgur.com/xvEKaDS.png)

Nice! Also looks close to what I see in most online visualizations. Here I'm using the same iterations-till-blowup value and then mapping that to a hue.

And now I'll leave you with the very high 12Kx10K resolution version of this image. And then we can talk about why that was a stupid idea. I won't render it on this page directly because that would tank its load times, so you'll have to click on [this link](https://i.imgur.com/HWGjDy4.jpg) to view it. If you compare it to the low res ones you should see the signifincantly greater amount of detail visible.

I wanted to see the tiniest details in the fractal pattern, and so I thought the way to go about doing that would be to create a very high resolution image. But there's only so much a tiny computer can do. And once you cross the screen resolutions currently available (4K is only 3840 x 2160, compared to the 12K x 10K image I generated which can never be fully represented on a 4K screen), you'll have to zoom in massively into the image to see details. So, while vieweing details, you'll never be able to see the full pictures.

So, apart from those reasons, why else was it a really stupid idea to create an extremely large, high resolution image?

- It takes forever.
- It still has an upper bound to the resolution.
- We're dealing with a fractal, we will never be able to probe its infinite depths by trying to create a tremendously large image and then zooming in.

So I decided to watch some more youtuve videos and figure out how to do this better. And I turned to Daniel Shiffman, one of the best youtube educators I know of in the programming space.

<iframe width="1440" height="619" src="https://www.youtube.com/embed/6z7GQewK-Ks" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Imma make a zoomer thingie.

## Zoomer

So, I fired up the p5 editor again. Since I was planning to keep the resolution fairly small, I figured I could keep things in the browser. Or rather, _because_ I wanted to keep it in the browser, I decided to go with the low low resolution of 400 x 400.

I also made it so the image can be zoomed using `+` and `-` and panned using the arrow keys. It still happens really slowly, but at least it works.

<iframe src="https://editor.p5js.org/scionofbytes/embed/kdAFrs4jI"></iframe>

I decided to get a little bit fancy and let the image appear iteration by iteration all the way up to 200. The iterations here is the maximum number of iterations at which we stop testing if the z value will grow beyond 2. As you can see, the more the iterations performed, the more detailed the picture. This effect is more apparent when you are zoomed into one of the spots. You can see the details becoming finer right in front of your eyes.
