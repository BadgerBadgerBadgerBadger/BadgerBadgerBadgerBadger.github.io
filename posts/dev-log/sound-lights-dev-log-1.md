---
title: 'Sound & Lights: Dev Log #1'
date: 
slug: ''
published: false

---
I recently began trying my hands at small electronics projects. I got myself a starter kit with an Arduino Uno, some wires, capacitors, resistors, and all the other little goodies they package in starter kits.

My initial goals were nebulous; I mainly wanted to try something new as lockdown started driving me up the wall. Somewhere along the way, I got the idea of using led strips to build an audio visualizer. It felt like a small enough project that I could tackle with my limited knowledge of electronics, and, ideally, learn a lot along the way.

This series of dev logs are supposed to chronicle that journey and help me fully absorb the concepts I'll learn along the way.

## The Plan

Though still somewhat nebulous, I knew I wanted something like a mountable board with LEDs attached to them. And I wanted these LEDs to react to music.

At first, I thought if it might be feasible to use a microphone module attached to the Arduino as the input sound source. But I quickly gave up on that idea. I wanted it to react to music and not the drilling my neighbor sometimes likes to do at odd hours.

I wondered if I should be sending a sound signal to the Arduino and have it analyze the spectrum for frequency bands and visualize that but gave up on that idea as well.

I finally settled on sending commands to the Ardunio to update the lights and leave all major processing work to whatever device would be sending those commands.

This was the first thing I got working.
