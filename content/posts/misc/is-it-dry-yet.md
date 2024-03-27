---
layout: post
title: Is it Dry Yet?
excerpt: "The one where I talk about my dirty laundry."
modified: 
categories: [misc]
tags: [home-automation, home-assistant, smart-plugs]
comments: true
share: true
---

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/61210f38-0a3a-40cd-854e-d38d7f6e685e)

> **Clarification**: It is literally about my laundry. I am not being metaphorical. I am not talking about my feelings or my secrets. I am talking about my clothes.  

## Intro

Our washing machine is a Samsung something-something that sends me a notification on the [SmartThings app](https://www.samsung.com/be/smartthings/app/) when it completes a cycle. This is useful since we almost always need to follow up a washing cycle with a drying one (or another load of laundry), and since it isn't a combi-washer-dryer, someone has to move the wet clothes from our smart washer to our dumb dryer.

The fact that our dryer is "dumb" is what led me to my automation journey. If it was Wifi-enabled and also sending out notifications as it completed a cycle, I wouldn't be writing this post. 

Most of the time it doesn't matter. Once the dryer is done drying, my girlfriend, or I, get to the clothes when we get to the clothes. It's not like it will undry itself. Sometimes, though, it does matter, for example, when the clothes need an extra drying cycle or when there are more wet clothes that are waiting in the washer. In these times it is useful to know if the dryer has finished a cycle.

And since it is not a smart device, I have to add the brains, somehow.

## Plan of Attack

My plan of attack is this:
- I have a [Shelly smart plug](https://www.shelly.com/en-be/products/shop/shelly-plus-plug-s) that I will put between Drew (the dryer is called Drew) and the wall.
- The plug will transmit power readings to my [Home Assistant](https://www.home-assistant.io/) setup.
- I will do..._something_ to detect when the power reading goes down from some high number to some very low number (I'm assuming it never goes to 0 since the little LED panel needs power).
- I will notify myself of this..._somehow_.

It's the _something_ and _somehow_ parts I am yet unsure of. My Home Assistant setup is as basic as it gets. I only started my home automation journey a little while ago and am unfamiliar with the possibilities. However, this feels like something that should be within the realm of my capabilities.

## Discovering Automations

The first thing I discovered as dug further into Home Assistant was [Automation](https://www.home-assistant.io/docs/automation/). In Home Assistant you create automations via the Automations system. You can turn on a light when a motion sensor senses someone in the room; you can turn on the vent when humidity reaches a certain level; you can make all the lights turn pink and romantic music play over the speakers when it is February 14th and your beloved comes home from work. The possibilities are _endless_.

## The Automation Setup

Figuring out the Automation setup I want was straightforward. There are 3 key components to the Home Assistant Automation system.
1. [Triggers](https://www.home-assistant.io/docs/automation/trigger/): The thing that kicks off an Automation. These can be of many types from a number on a sensor changing, to an incoming MQTT payload.
2. [Conditions](https://www.home-assistant.io/docs/automation/condition/): An optional extra check to ensure that when the Automation triggers everything is in the state you want it to be. You can check the value of sensors, check the on/off status of devices, or even [check if the sun is up](https://www.home-assistant.io/docs/scripts/conditions/#sun-condition).
3. [Actions](https://www.home-assistant.io/docs/automation/action/): This where we _do_ something in our Automation. This is where you will want to set up that electric rooster crowing when the sun comes up (if you want to build your own weird alarm clock for some reason), or send a message on a webhook when the kids are using too much electricity.

Thinking over my needs I decided on these things for my Trigger/Condition/Action setup:
1. **Trigger**: I want the Automation to start when Drew's power draw falls below 10 W (as reported by Sheila, the smart plug), and has been doing so for more than a minute.
2. **Condition**: I want to ensure this Automation only fires if we have consumed more than a 100 Watts of power sometime in the last 5 minutes.
3. **Action**: Inform me somehow that this has happened.

This is what tht

At first, I wasn't entirely sure how to notify myself. I looked into [Home Assistant's Restful Notifications](https://www.home-assistant.io/integrations/notify.rest/) but could not make it work for the life of me. Then I decided to check if I could send an email. I found [Home Assistant's SMTP Notifications](https://www.home-assistant.io/integrations/smtp/) and decided to give it a go. Again, I could not make it work. It asked me to enter my Gmail password in the Home Assistant configuration file, but I was not comfortable doing that. I decided to look for another way.

I finally stumbled upon the [Home Assistant iOS App](https://www.home-assistant.io/integrations/ios/) which I installed it on my phone and pointed to my Home Assistant server. This allowed me to send notifications to my phone (my phone became available as a target for notifications).

## Exposing to the World

I should mention that my Home Assistant server was not available to the internet at that point. All integrations were over local wifi with firewall rules to prevent any external access. I did not have to expose my Home Assistant server to the internet to get notifications on my phone since they were both part of a [Tailnet](https://tailscale.com/kb/1136/tailnet). This meant that as long as I had the [Tailscale App](https://tailscale.com/download/ios)  on my phone and Tailscale VPN enabled, I could access my Home Assistant server from anywhere in the world.

And this would have been good enough for me, but my girlfriend also needs notifications on _her_ phone and asking her to install Tailscale and turn it on/off _before_ she could access the notifications was a bit much.

So I decided to expose my Home Assistant server to the internet.

My first attempt was using Let's Encrypt and DuckDNS following a guide (or three) similar to [this](https://www.makeuseof.com/access-home-assistant-server-remotely-duckdns-letsencrypt/). I won't go into all the details, but I will say that it was a _pain_. Ultimately, after struggling for several days, I gave up. I gave up and decided to go back to my girlfriend, head bowed, and give her the bad news.

But then I stumbled upon [Tailscale Funnels](https://tailscale.com/kb/1223/funnel). Since I was using Tailscale anyway, with a single additional invocation, I can have a secure, encrypted, and authenticated tunnel to my Home Assistant server. 
```sh
➜  ~ tailscale funnel --https=8443 --bg 8123
```
No extra DNS required since I can use my Tailnet's name and no extra certificates required since Tailscale handles all of that for me.

The only caveat to this approach is that Tailscale does not yet support domain name handling. So I can't assign a nice domain name to my Home Assistant server. Furthermore, I only have [3 ports available to me](https://tailscale.com/kb/1223/funnel#limitations). This means that if I were to have other services on that machine that I want to expose to the world, I can at most have 2 more and they will have to be referenced by their port numbers.

This is something I will have to come back to later, but for now both me and my girlfriend have access to our Home Assistant server from anywhere in the world and are happy ducks.
