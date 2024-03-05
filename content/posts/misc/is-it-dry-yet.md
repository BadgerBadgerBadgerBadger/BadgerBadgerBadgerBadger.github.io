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

The fact that our dryer is "dumb" is important. If it was Wifi-enabled and also sending out notifications as it completed a cycle, I wouldn't be writing this post. 

Most of the time it doesn't matter. Once the dryer is done drying, my girlfriend or I get to the clothes when we get to the clothes. It's not like it will undry itself. Sometimes, though, it does matter. Mainly when the clothes need an extra drying cycle or when there are more wet clothes that are waiting in the washer. In these times it is useful to know if the dryer has finished a cycle.

And since it is not a smart device, I have to add the brains, somehow.

## Plan of Attack

My plan of attack is this:
- I have a [Shelly smart plug](https://www.shelly.com/en-be/products/shop/shelly-plus-plug-s) that I will put between Drew (the dryer) and the wall.
- The plug will transmit power readings to my [Home Assistant](https://www.home-assistant.io/) setup.
- I will do..._something_ to detect when the power reading goes down from some high number to some very low number (I'm assuming it never goes to 0 since the little LED panel needs power).
- I will notify..._somehow_.

It's the _something_ and _somehow_ parts I am yet unsure of. My Home Assistant setup is as basic as it gets. I only started my home automation journey a little while ago and am unfamiliar with the possibilities. However, this feels like something that should be within the realm of my capbilities.

## Discovering Automations

The first thing I discovered was [Automation](https://www.home-assistant.io/docs/automation/). In Home Assistant you create automations via the Automations system. You can turn on a light when a motion sensor senses someone in the room, you can turn on the vent when humidity reaches a certain level, or you can make all the lights turn pink and romantic music play over the speakers when it is February 14th and your beloved comes home from work. The possibilities are _endless_.

Or...not quite endless.

## First Hurdle

Figuring out the Automation setup I want was straightforward. There are 3 key components to the Home Assistant Automation system.
1. [Triggers](https://www.home-assistant.io/docs/automation/trigger/): The thing that kicks off an Automation. These can be of many types from a number on a sensor changing to an incoming MQTT payload.
2. [Conditions](https://www.home-assistant.io/docs/automation/condition/): An optional extra check to ensure that when the Automation triggers everything is in the state you want it to be. You can check the value of sensors, check the on/off status of devices, or even [check if the sun is up](https://www.home-assistant.io/docs/scripts/conditions/#sun-condition).
3. [Actions](https://www.home-assistant.io/docs/automation/action/): This where we _do_ something in our Automation. This is where you will want to set up that electric rooster crowing when the sun comes up (if you want to build your own weird alarm clock for some reason), or send a message on a webhook when the kids are using too much electricity.

Thinking over my needs I decided on these things for my Trigger/Condition/Action setup:
1. **Trigger**: I want the Automation to start when Drew is pulling less than 10 Watts of power (as reported by Sheila, the smart plug), and has been doing so for more than a minute.
2. **Condition**: I want to ensure this Automation only fires if we have consumed more than a 100 Watts of power sometime in the last 5 minutes.
3. **Action**: Inform me somehow that this has happened.

