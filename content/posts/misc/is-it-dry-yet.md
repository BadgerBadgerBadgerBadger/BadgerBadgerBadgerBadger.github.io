---
layout: post
title: Is it Dry Yet?
excerpt: "It is, in fact, about drying clothes."
modified: 2024-03-07T00:00:00+0:00
categories: misc
tags: [home-automation, home-assistant, smart-plugs]
comments: true
share: true
---

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/61210f38-0a3a-40cd-854e-d38d7f6e685e)

## Intro

Our washing machine is a Samsung something something that sends me a notification on the SmartThings app when it completes a cycle. This is useful since we almost always need to follow up a washing cycle with a drying one, and since it isn't a combi washer-dryer, someone has to move the wet clothes from our smart washer to our dumb dryer.

The fact that our dryer is "dumb" is important. If it was wifi-enabled and also sending out notifications as it completed a cycle, I wouldn't be writing this post. 

Most of the time it doesn't matter. Once the dryer is done drying, my girlfriend or I get to the clothes when we get to the clothes. Sometimes, though, it does matter. Most often when the clothes need an extra drying cycle or when there's more wet clothes that are waiting in the washer. In these times it is useful to know if the dryer has finished a cycle.

And since it is not a smart device, I have to add the brains, somehow.

## Plan of Attack

My plan of attack is this:
- I have a Shelly smart plug that I will put between the dryer and the wall.
- The plug will be transmitting power readings to my Home Assistant setup.
- I will do..._something_ to detech when the power reading goes down from some high number to some very low number (I'm assuming it never goes to 0 since the little LED panel needs power).
- I will notify..._somehow_.

It's the _something_ and _somehow_ parts I am yet unsure of. My Home Assistant setup is as basic as it gets. I only started my home automation journey a little while ago and am unfamiliar with the possibilities. However, this feels like something that should be within the realm of possibility.
