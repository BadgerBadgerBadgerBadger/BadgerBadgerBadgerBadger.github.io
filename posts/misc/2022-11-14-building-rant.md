---
layout: post
title: Building Rant
excerpt: "A fun little service I wrote to help me rant better."
modified: 2022-11-14T21:13:25-04:00
categories: misc
tags: [golang, humor, intepreters]
comments: true
share: true
---

## Intro

Sometime in mid to late 2021 (which is a period I've entirely made up because I don't _actually_ remember when any of this happened), I needed to rant on my company Slack. Something had happened to ruin my day and I needed to get my feelings out.

But ranting takes more effort than you'd think. If you want your rant to have _impact_ there needs to be just the right amount of `!`s in just the right places. Too little, and your words may as well be the kisses of tiny wind fairies for all the notice they'll get. Too many and you're overwhelming the fundamental message of the rant. Similarly a `!` where it doesn't belong is a crime against rehetoric. You don't want people to lose the flow of your beautiful ranty sentences and focus overmuch on the `!`s.

Moreover, in this age of "show don't tell", emojis have a greater impact than even words. And they too require injection in all the right places.

So yeah, ranting takes effort. And the problem with effort is that you cannot spill your guts in a stream-of-conciousness rant _and_ format it to have the right impact all at the same time. Try to inject `!`s or emojis in your rants, and if you want to do a good job of it, you _will_ have to calm down, you _will_ have to think it through. Rare is the person who can both give in to their inner child and throw a text tantrum while simultenously channeling their inner Millenial teen.

Enter, Rant! a toy project to help me, well, rant!

Rant! reads in my ranty text, whether it be a line or a paragraph and injects `!`s and emojis in all the right places. To say that Rant! has taken my ranting to the next level would be an understatement. While before my rants barely conveyed the depth of my outrage, now they scream themselves out to the world. No, really, all the letters come out in uppercase.

For example, if I wanted to rant the following rant:

```
Chinese demand is raising the price. I TOLD YOU SO! Only I can say horrible untrue things about me, it may be an open blank. Please send me flowers & a total bust!
```

I _could_ just post it into Slack, or I could invoke Rant! to do it for me!

```
CHINESE DEMAND IS RAISING THE PRICE!!! 😠 I TOLD YOU SO!!! 🤬  ONLY I CAN SAY HORRIBLE UNTRUE THINGS ABOUT ME, IT MAY BE AN OPEN BLANK!!! 😡 PLEASE SEND ME FLOWERS & A TOTAL BUST!!! 😤
```

So how did I go about building Rant!?

## You Never Know

You never know when a random thing that you learned could come in use. A long time back I remember going through [Robert Nystrom's Crafting Interpreters](https://craftinginterpreters.com/) blog/book. I never got past the lexing and parsing stages, but it turns out that was enough to help me when I wanted to build Rant!

Crafting Interpreters was my first introduction to the world of programming language design and implementation. I learned what it means to lex tokens and parse them into an AST. I learned...well, not much beyond this, but as I've already mentioned, just knowing the lexing and parsing stages was all I needed to get working on Rant! In fact, I didn't even need to know the parsing stage, being able to lex a source string into tokens was enough.

The long-winded point I'm trying to make is that you never know what could end up being useful. So learn random things.

I started with a few rules in mind for what I wanted to happen when I used Rant! Some of these things in no particular order are:
- All text should be in upper case. An input of `I'm so mad` should give an output of `I'M SO MAD`
- When a sentence ends with a period, I wanted to replace that with three (THREE, I tell you!!!) `!`s and an angry emoji. Given `I'm so mad.`, I wanted `I'M SO MAD!!! 😡`
- When a sentence ended with a `?`, I wanted it replaced by a `?!` and an angry emoji. I think you're startingt to see the pattern.
- Whenever either a single `!`, a double `!!` or a triple `!!!` is encountered, replace them all with a triple `!!!`s and an angry emoji. Any sequence of `!`s beyond that should be preserved as is.

I'm not sure if I've covered all the rules here but it should be sufficient to paint a picture.

## Scanning/Lexing/Tokenization

First scan the input string using the lexing approach Robert talks about in the [Scanning](https://craftinginterpreters.com/scanning.html) chapter of his excellent book. I convert the string into tokens that I can then operate on. Once I have tokens such as `Bang`, `Period`, `BangBang`, etc. I can loop over them and either print them as is (like the case for the `Dot` token which represents a `.` without any preceeding or succeeding space, indicating it's probably not being used as a period at the end of a sentence), or replaced by something more exciting (such as a `Bang` token being replaced by `!!!` + `<random-angry-emoji`>).

Honestly, that's it, that's the bulk of the ranting logic. There's code in the repo (check the end of the post) that performs the Slack integration, Oauth and a web API, but none of that is really that interesting.

The final interesting bit is not wanting to return the same random emoji twice in a row. No one wants to see two 😡s one after the other. So I keep track of the emojis returned by the random emoji method and remove it from the pool of potential emojis. I'm sure the algorithm I use can be improved upon, but it does what I need it to and I'm not worrying about performance at the moment.

## The Code

You can find the code on [Github](https://github.com/BadgerBadgerBadgerBadger/rant) and maybe [give it a try](https://rant.badgerbadgerbadgerbadger.dev/) and give me ideas for improvement. Hope you enjoyed reading about this fun little project.
