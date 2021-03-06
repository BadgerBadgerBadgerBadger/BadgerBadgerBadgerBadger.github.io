---
layout: post
title: "Styleguides: Hapi Vs. Airbnb"
excerpt: "Comparing the styling rules of two very popular entities and describing why I think Hapi sucks and Airbnb rules!"
modified: 2016-12-04T14:17:25-04:00
categories: code-craftsmanship
tags: [nodejs, coding-style, rant]
comments: true
share: true
---

This post was inspired by my writing of another post which you can read [here]({{ site.baseurl }}{% post_url 2016-12-01-letter-casing-in-names-of-imported-nodejs-modules %}). On that one I talk about letter casing conventions in the names of imported NodeJS modules. Not the catchiest of titles but if it's something that interests you, have a look.

I'd been a strict adherer of the [Airbnb javascript styleguide](https://github.com/airbnb/javascript) for more than a year. In the previous company I'd worked at, we needed a styleguide that made sense, was intuitive, and made reading our code easier. So we did our research, found the airbnb styleguide to be the most suitable, forked it from airbnb's repo, made some modifications to suit our usecases (while retaining over 95% of the original rules), and moved on with our lives, worrying less about the style and more about our product.

Airbnb's is the kind of styleguide that let's you do that. Once you've gone through it, understood the reasoning behind the rules and worked within their bounds for a few weeks, they become second nature. Mostly because most of the rules make intuitive sense in the first place. You quickly find yourself focusing less on the style and more on the meat of the code, because the style has already become second nature.

The company I'm working at right now uses a subset of [Hapi](http://hapijs.com/)'s [styleguide](http://hapijs.com/styleguide). At first I was excited to be practicing code in a new style. Every new style is an opportunity to enter the minds of the people who wrote it and gain a new perspective. And every new perspective improves us in subtle ways.

And the Hapi styleguide *did* do that for me. I came across the PascalCase naming convention, realized the reasoning behind it, and wondered why I hadn't used it before when it made [so much sense](https://scionofbytes.github.io/code-craftsmanship/letter-casing-in-names-of-imported-nodejs-modules/#project-modules).

But here's the thing: I had to figure out the reasoning behind it myself. The Hapi styleguide did not do that for me.

The entire tone of the Hapi styleguide reads like a list of commandments. *Do this*, *Don't do that*, *This is not allowed.* I feel like I'm being dictated to and not taught. There are plenty of *rules* but no *reasoning*.

If you compare this to the airbnb styleguide, you'll find that most of their rules come with some sort of reasoning behind it. *Do this. Why? Because this leads to this and thus you'll experience this benefit.* That's the entire tone of the airbnb styleguide- one of teaching and building intuition.

Try this. Open up each styleguide and look for the word *why*. I did this in order to quickly test out how often airbnb explains itself and how often Hapi does the same. For airbnb I got 53 hits on doing a search in Chrome. For Hapi I got 0.

Styleguides exist to enforce consistency across a codebase. If all your engineers are following the same styleguide, your code will look the same all over the place. This makes it easier for the entire company to work on the code and for one person to contribute to another's work.

But a good styleguide also introduces patterns that improve your thinking and the architecture of your code. It makes you question why certain things should be done a certain way and come up with answers that make the most *sense*. It makes you a better engineer in the long run.

And that's all I have to say on that.
