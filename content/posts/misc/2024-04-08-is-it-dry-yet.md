---
layout: post
title: Is it Dry Yet?
excerpt: "The one where I talk about my dirty laundry."
image: https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/7b71e898-f7bc-4d72-91e6-27fd20a74a88
modified: 2024-04-08T00:00:00+01:00
categories: [misc]
tags: [home-automation, home-assistant, smart-plugs]
comments: true
share: true
---

> **Clarification**: This is literally about my laundry. I am not being metaphorical. I am not talking about my feelings or my secrets. I am talking about my clothes.  

## Intro

Our washing machine is a Samsung something-something that sends me a notification on the [SmartThings app](https://www.samsung.com/be/smartthings/app/) when it completes a wash cycle. This is useful since we almost always need to follow up a washing cycle with a drying one and maybe even another load of laundry. The machine isn't a combined-washer-dryer so someone has to move the wet clothes from our smart washer to our dumb dryer.

The fact that our dryer is "dumb" is what led me to my automation journey. If it was Wifi-enabled and also sending out notifications as it completed a cycle, I wouldn't be writing this post. 

Having been spoiled by knowing exactly when my washing has completed (so I know when to go move things to the dryer) I decided to do something similar for the dryer. This is especially useful, for example, when the clothes need an extra drying cycle or when there are more wet clothes that are waiting in the washer. In these times it is useful to know if the dryer has finished a cycle.

And since it is not a smart device, I have to add the brains, somehow.

## Plan of Attack

This is how I decided to approach the problem:
- I have a [Shelly smart plug](https://www.shelly.com/en-be/products/shop/shelly-plus-plug-s) that I would put between Drew (the dryer is called Drew) and the wall.
- The plug would transmit power readings to my [Home Assistant](https://www.home-assistant.io/) setup.
- I would do _something_ to detect when the power reading goes down from some high number to some very low number (I'm assuming it never goes to 0 since the little LED panel needs power).
- I will notify myself of this _somehow_.

It's the _something_ and _somehow_ parts I was unsure of. My Home Assistant setup is as basic as it gets. I only started my home automation journey a month or two ago and was unfamiliar with the possibilities. However, this felt like something that should be within the realm of HASS's capabilities.

## Discovering Automations

The first thing I discovered as I dug further into Home Assistant was [Automation](https://www.home-assistant.io/docs/automation/)s. In Home Assistant you create automations via the Automations system. You can turn on a light when a motion sensor senses someone in the room; you can turn on the vent when humidity reaches a certain level; you can make all the lights turn pink and romantic music play over the speakers when it is February 14th and your beloved comes home from work. The possibilities are _endless_.

## The Automation Setup

Figuring out the Automation setup I wanted was straightforward. There are 3 key components to the Home Assistant Automation system.
1. [Triggers](https://www.home-assistant.io/docs/automation/trigger/): The thing that kicks off an Automation. These can be of many types ranging from a number on a sensor changing, to an incoming MQTT payload.
2. [Conditions](https://www.home-assistant.io/docs/automation/condition/): An optional extra check to ensure that when the Automation triggers everything is in the state you want it to be. You can check the value of sensors, check the on/off status of devices, or even [check if the sun is up](https://www.home-assistant.io/docs/scripts/conditions/#sun-condition).
3. [Actions](https://www.home-assistant.io/docs/automation/action/): This where we _do_ something in our Automation. This is where you will want to set up that electric rooster crowing when the sun comes up (if you want to build your own weird alarm clock for some reason), or send a message on a webhook when the kids are using too much electricity.

Thinking over my needs I decided on the following for my Trigger/Condition/Action setup:
1. **Trigger**: I want the Automation to start when Drew's power draw falls below 10 W (as reported by Sheila, the smart plug), and has remained under 10 W for more than 5 minutes.
2. **Condition**: I want to ensure this Automation only fires if we have consumed more than a 100 Watts of power sometime in the last 5 minutes.
3. **Action**: Inform me somehow that this has happened.

While the Trigger and Condition were straightforward, the Action was a bit more nebulous. At first, I wasn't entirely sure how to notify myself. I looked into [Home Assistant's Restful Notifications](https://www.home-assistant.io/integrations/notify.rest/) but could not make it work for the life of me. Then I decided to check if I could send an email. I found [Home Assistant's SMTP Notifications](https://www.home-assistant.io/integrations/smtp/) and decided to give it a go. Again, I could not make it work. It asked me to enter my Gmail password in the Home Assistant configuration file, but I was not comfortable doing that. I decided to look for another way.

I finally stumbled upon the [Home Assistant iOS App](https://www.home-assistant.io/integrations/ios/) which I installed on my phone and pointed to my Home Assistant server. This allowed me to send notifications to my phone (my phone became available as a target for notifications).

The full Automation setup looks like this:

```yaml
alias: Is it dry yet?
description: Notifies when Drew (the dryer) has finished a drying cycle.
trigger:
  - id: Sheila Power Below 10 W
    platform: numeric_state
    entity_id:
      - sensor.sheila_power
    below: 10
    for:
      minutes: 5
condition:
  - condition: numeric_state
    entity_id: sensor.sheila_power_5_minutes_ago
    above: 100
action:
  - service: notify.mobile_app_sharpie
    data:
      message: Drew has finished a cycle. You should unload him.
      title: Dryer Cycle Finished
  - service: notify.mobile_app_buttercups_android
    data:
      message: Drew has finished a cycle. You should unload him.
      title: Dryer cycle finished.
  - service: media_player.play_media
    target:
      entity_id: media_player.thuis
    data:
      announce: true
      media_content_type: mp3
      media_content_id: https://<static-host>/drew-finished-drying.mp3
mode: single
```

`notify.mobile_app_sharpie` and `notify.mobile_app_buttercups_android` are the entities exposed by the Home Assistant iOS and Android apps on my phone and my girlfriend's phone respectively. `media_player.thuis` is the entity exposed by my Google Nest speaker group. Not being simply content to send a notification to my phone, I also wanted to play a sound on my Google Nest speaker group. The sound is a mp3 file that I host on a static host.

`sensor.sheila_power` is the sensor entity exposed by my smart plug and what lets me monitor the power draw of the dryer. `sensor.sheila_power_5_minutes_ago` is a sensor that I created using the [SQL Integration](https://www.home-assistant.io/integrations/sql). I created a custom sensor to get the power draw 5 minutes ago.

```sql
select *
from states
where metadata_id = (select states_meta.metadata_id
                     from states_meta
                     where states_meta.entity_id = 'sensor.sheila_power')
  and datetime(last_updated_ts, 'unixepoch') < datetime('now', '-5 minutes')
order by last_updated_ts desc
limit 1
```

In combination this ensures that this Automation only triggers when our power draw falls below 10 W for at least 5 minutes, and we have consumed more than 100 W of power just before. This gives me a good indication that the dryer has finished a cycle and I should go unload it.

## Exposing to the World

I should mention that my Home Assistant server was not available to the internet at that point. All integrations were over local Wi-Fi with firewall rules to prevent any external access. I did not have to expose my Home Assistant server to the internet to get notifications on my phone since they are both part of a [Tailnet](https://tailscale.com/kb/1136/tailnet). This meant that as long as I had the [Tailscale App](https://tailscale.com/download/ios)  on my phone and Tailscale VPN enabled, I could access my Home Assistant server from anywhere in the world.

And this would have been good enough for me, but my girlfriend also needs notifications on _her_ phone and asking her to install Tailscale and turn it on/off before she could access the notifications was a bit much.

So I decided to expose my Home Assistant server to the internet.

My first attempt was using Let's Encrypt and DuckDNS, following a guide (or three) similar to [this](https://www.makeuseof.com/access-home-assistant-server-remotely-duckdns-letsencrypt/). I won't go into all the details, but I will say that it was a _pain_. Ultimately, after struggling for several days, I gave up. I gave up and decided to go back to my girlfriend, head bowed, and give her the bad news.

But then I stumbled upon [Tailscale Funnels](https://tailscale.com/kb/1223/funnel). Since I was using Tailscale anyway, with a single additional invocation, I could have a secure, encrypted, tunnel to my Home Assistant server. 
```sh
➜  ~ tailscale funnel --https=8443 --bg 8123
```
No extra DNS required since I can use my Tailnet's name and no extra certificates required since Tailscale handles all of that for me.

The only caveat to this approach is that Tailscale does not yet support domain name handling. So I can't assign a nice domain name to my Home Assistant server. Furthermore, I only have [3 ports available to me](https://tailscale.com/kb/1223/funnel#limitations). This means that if I were to have other services on that machine that I want to expose to the world, I can at most have 2 more, and they will have to be referenced by their port numbers.

This is something I will have to come back to later, but for now both me and my girlfriend have access to our Home Assistant server from anywhere in the world and are happy ducks.

Of course, I limited her account to only have access to the notifications and some views. I don't want her messing with my automations and I don't want a third party poking around my smart home setup.
