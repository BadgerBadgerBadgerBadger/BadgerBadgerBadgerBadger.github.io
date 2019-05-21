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

Let's say we have a grid of pixels, and each pixel can be represented by coordinates. Let's say also that we have a function (the programming kind, not the math kind) `testMandelbrot` which takes a coordinate `c` and tells us if `c` is in the Mandelbrot Set or not. That function and most of the math we need to perform is in [this file](https://github.com/ScionOfBytes/smooth-mandelbrot/blob/master/js/compy-stuff.js#L20). As you can see, it requires very little code.

For a proper understanding of why all this math is happening (and especially if my attempt to explain wasn't sufficient), have a look at these videos.

1. [The Mandelbrot Set - Numberphile](https://www.youtube.com/watch?v=NGMRB4O922I&t=16s) - More about the Mandelbrot Set.
1. [Mandelbrot Set on Vsauce](https://www.youtube.com/watch?v=MwjsO6aniig) - A very entertaining and education video by the beloved Michael Stevens.

And then read this excellent little page by Karl Sims [Understanding Julia and Mandelbrot Sets by Karl Sims](https://www.karlsims.com/julia.html) - this one helped me a _lot_.

## Early Attempts

My first attempt was semi-successful. I made many mistakes, wrote a lot of messy code, and managed to get something going that rendered very slow even for a 450 x 200 resolution canvas. You can find it [here](https://editor.p5js.org/scionofbytes/full/UfCfqKVrY). If you play around with the resolution, you will notice that upping it to something still small like 600 x 350 will make it render significantly more slower than previously. This is no surprise. It might only be an increase of 200 pixels horizontally and 150 pixels vertically, but overall there are now 30K more pixels to scan than before.

I also have a version of this with cleaner code on https://openprocessing.org ([this one](https://www.openprocessing.org/sketch/707203)), but I stressed it out too much in terms of resolution and [openprocessing](https://openprocessing.org) is taking too long to render it.

This is how it looks, by the way:

![p5 editor result](https://i.imgur.com/rKzuPXj.png)

I decided it was running slow because it was in Javascript. I am ashamed to say that even after five years of working in the industry I still have such naive thoughts. The truth was my code was merely bad and lacking any sort of polish or optimisation. 

So the next step of my plan was to try to do the same thing in [Processing 3](https://processing.org).

I started small:

![kotlin version low res](https://i.imgur.com/QyoLM9U.png)

And then went a tiny bit bigger.

![kotlin version higher res](https://i.imgur.com/OxFtnsN.png)

These took way too long to generate, though, the first one clocking in at 2251ms, the second at 7869ms.

I was still making mistakes here. I was using a full package to do the complex math and several other overly complicated things. There wasn't a noticeable performance gain over the JS code.

And then I decided to do something really stupid. I wanted to see a highly zoomed in version of the Mandelbrot Set and admire the beautiful fractal patterns so I thought, why not make a _really_ large image and then zoom into that using a regular image viwere and admire the fractals that way?

So I tried to generate a 12000x10000 image. That's 120 million pixels. A 120 mega-pixel image. I tried to generate that on my laptop. And I made a royal mess of it, too. Not only did it take over an hour to do, I forgot to save it, so just to show you guys that image I'll have to let my laptop run hot for an hour, again, and it won't even be that great an image. You'll get to see it anyway.

But let's take a step back.

I didn't want to just make a 120 megapixel image. I wanted to make a 12 gigapixel one. But the moment I tried to allocate an array that large JVM told me to stick my head where the sun don't shine. Okay, 12K x 10K it would be. I wanted this image to look extra special so I decided to make some smaller ones look really good.

First a 1200 x 1000 that took 38,785ms:

![kotlin version 1200x1000](https://i.imgur.com/nI51GLq.png)

This is pretty nice looking, actually, but let's see if we can do something prettier.

![kotlin version 1200x1000 with extra greys](https://i.imgur.com/fHBJM8q.png)

For each of the points _not_ inside the Mandelbrot set, I'm taking the number of iterations it took before it blew up and mapping that from 0 - 200 (since 200 is the maximum number of iterations I'm checking) to 0 - 255 (8 bit greyscale color range).

Now we can not only see the scattered Mandelbrot islands, we can also see the connections between them. These connections are too thin to show up on such a low resolution image, but the surrounding shading gives them away.

Let's try something in another color.

![kotlin version 1200x1000 with red borders](https://i.imgur.com/d0Mt31G.png)

For this I decided to color the inside of the Set fully black, and let the outside be mapped from 0 (full black) to #FF0000(full red), giving us a vivid red border.
 
Let's do something even more colorful.
 
![kotlin version 1200x1000 with multi-color borders](https://i.imgur.com/xvEKaDS.png)

This looks closer to what I see in most online visualisations. Here I'm using the same iterations-till-blowup value and then mapping that to a hue (think [HSL](https://en.wikipedia.org/wiki/HSL_and_HSV)).

Now I'll leave you with the 12K x 10K resolution version of this image. I won't render it on this page directly because that would tank its load time. You'll can click on [this link](https://i.imgur.com/HWGjDy4.jpg) to view it. If you compare it to the low res ones you should see the signifincantly greater amount of detail visible.

## A More Sensible Approach

I wanted to see the tiniest details in the fractal pattern, and I thought the way to go about doing that would be to create a very high resolution image. But there's only so much a tiny computer can do. And once you cross the screen resolutions currently available (4K is only 3840 x 2160, compared to the 12K x 10K image I generated which can never be fully represented on a 4K screen), you'll have to zoom in a lot to see details. So, while vieweing details, you'll never be able to see the full picture, anyway.

Then there's the matter of how long it takes to generate one of these. If you want to play around with iteration values or colors or any other paramter, you would have to generate the static image all over again. Not fun.

I decided to watch some more youtuve videos and figure out how to do this better. I turned to Daniel Shiffman, one of the best youtube educators I know of in the programming space.

<iframe width="1440" height="619" src="https://www.youtube.com/embed/6z7GQewK-Ks" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

A new plan formed in my mind.

## Zooming and Panning

I fired up the p5 editor again. Since I was planning to keep the resolution fairly small, I figured I could keep things in the browser. Or rather, _because_ I wanted to keep it in the browser, I decided to go with the low low resolution of 400 x 400.

I also made it so the image can be zoomed using `+` and `-` and panned using the arrow keys. It still happens really slowly, but at least it works.

<iframe src="https://editor.p5js.org/scionofbytes/embed/kdAFrs4jI"></iframe>

I decided to get a little bit fancy and let the image appear iteration by iteration all the way up to 200. The iterations here is the maximum number of iterations at which we stop testing if the z value will grow beyond 2 (blow up). As you can see, the more the iterations performed, the more detailed the picture. This effect is more apparent when you are zoomed into one of the spots. You can see the details becoming finer right in front of your eyes.

Zooming and panning is indeed the way to go. If you are to look at only a certain part of the image at a time, why bother rendering anything else. Graphics programmers have known of this since ages beyond reckoning. It took me a failed attempt at generating a high res image at speed to think of doing the same thing (and ideas from Shiffman's video).
