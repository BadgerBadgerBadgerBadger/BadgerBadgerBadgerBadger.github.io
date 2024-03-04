---
layout: post
title: Short Tale of Intel -> Apple Silicon Migration
excerpt: A few notes about my experience getting my hands on an Apple Silicon Mac and migrating my stuff from my old Intel Mac.
modified: 2024-03-13T00:00:00+01:00
categories: misc
tags:
  - apple
  - story
comments: true
share: true
---

I recently got handed a new macbook at work. It's a MacbookPro with an M3 chip and my first live encounter with Apple Silicon. I have been hearing many good things about Apple Silicon and having experienced it for myself for the past day I must say my old Intel Macbook can't compare. I also know that comparing my 2020 Intel Macbook which has been used and abused for 4 years to a brand new, out-of-the-box 2023 M3 Mac is not an [Apples to Apples comparision](https://media.tenor.com/a2gst_5S-RAAAAAi/uarrr-finger-guns.gif). So I will take the word of tech reviewer through the years and my own biased experience of not having to wait seconds for compilations which now happen in subsecond durations.

The point of this post is to talk about the challenges I faced getting up and running after moving my stuff from an Intel Mac to an Arm Mac.

## Migration Assistant

While not specific to the Intel -> Arm transfer, I was reminded again that I should probably be running Migration Assistant over a lightning cable and not Wifi. It took all night and the way the numbers kept going up sometimes, instead of down, gave me a ton of anxiety. The next time I do this I have to remind myself that taking 5 extra minutes to get that cable out and connected will save me _hours_ later.

## It Does Not Start with Rosetta

I'm not entirely sure if this is always the case or if it ended up being the case becase I performed a Migration, but [Rosetta](https://support.apple.com/en-us/102527) was not already installed on my system. Instead, any time I tried to run a process that was Intel based, I got a pop-up that asked me if I want to install Rosetta.

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/2e55563b-7f61-4ad9-aeb9-1ff42469ff98)

I put it off for as long as I could. I wanted to have everything running as natively Apple Silicon as I could possibly manage. I finally did cave when I discovered a few tools did not have an Arm counterpart, but I think I managed to reinstall a lot of things before I hit that stage.

## You Can See What's Running on Rosetta

If you open the [Activity Monitor](https://support.apple.com/guide/activity-monitor/welcome/mac), you can [check which apps are running on Rosetta](https://thenextweb.com/news/how-to-check-app-running-m1-native-version-on-mac). In the CPU tab, look at the Kind column. It will either say _Intel_ or _Apple_.

I managed to discover several things running on Rosetta and then went and found Apple Silicon versions of them.

## Easier Than I Imagined

I talked to a colleague who got his Apple Silicon Mac more than three years ago when these things were fresh out of the oven. His tales of trial and tribulations made me grateful that I am making the move now when so many things have adopted Apple Silicon and have Arm versions released. I code primarily in Golang and it was almost trivial to clean up my old Golang versions and install Apple Silicon compatible ones.

## Wrapping it Up

I started a draft for this post when I began my migration journey. I imagined I would have to go through a lot more trouble than I actually ended up going through. Almost every app had an Apple Silicon version that was a simple download+replace operation and the story held true for nearly everything I needed to install via brew. I was up and running in a matter of hours.

I thank everyone who have been experimenting with Apple Silicon these last few years. They walked with hobbling steps, so people like me could run.
