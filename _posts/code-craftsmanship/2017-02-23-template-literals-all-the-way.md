---
layout: post
title: "Template Literals, All the Way!"
excerpt: "Should you use ES6 template literals all the time? Let's find out."
modified: 2017-02-23T14:17:25-04:00
categories: code-craftsmanship
tags: [javascript, nodejs, coding-style]
comments: true
share: true
---

Today, while at work, I had to turn a single-quoted string into a [template literal](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals) because mid-way through the string I realized I had to put in an apostrophe. This has happened often enough that I want to stop thinking in this dual single-quoted/template-literal mode and just go template-literal all the time.

Now before I say anything more I'd like to clarify that I'm working with NodeJS almost all the time with full feature support for template strings and **no** issues with things like minification. So anything in my research that had to do with `not supported in all environments` was discarded as invalid.

After an initial bout of googling I didn't find much to deter me from using template literals.

[This](http://softwareengineering.stackexchange.com/a/309623/199195) SO answer speaks of (among other things) IDE support and how having it makes working with template literals easier coz placeholders get highlighted. The [Airbnb Style Guide](https://github.com/airbnb/javascript#strings) prefers single-quoted strings and using template literals for interpolation. It doesn't say we *shouldn't* use template literals all the time. And then there's [this thread on reddit](https://www.reddit.com/r/javascript/comments/52ions/your_opinion_on_using_backtick_as_the_default/) that leads to a bunch of different places, most of which talk about browser support and minification and stuff that doesn't matter in my controlled NodeJs environment. 

[This guy](https://www.reddit.com/r/javascript/comments/52ions/your_opinion_on_using_backtick_as_the_default/d7otae8/) ran tests to figure out if using template literals poorly impacts performance but according to the results template literals are actually faster.

So I decided to see if there was any other research around performance impact. One would assume that template literals would be slower than a plain old string (when not interpolating) because a plain old string is just a plain old string but a template literal would check for interpolation even when no placeholders exist (though a good compiler would account for this and optimize accordingly).

Someone did a test of ES6 features on a bunch of platforms (including Node v6.9.3) and the [results](https://kpdecker.github.io/six-speed/) don't tell me anything about single quoted strings vs. template literals in a non-interpolating string (in general the results don't look very promising for ES6 features, though; eek!). 

Most research seems to be centered around whether it's more performant to concatenate strings or use template literals when interpolation is required. No one seems to have a strong stance on whether template literals should or should not be used for everything.

Some arguments I came across for why they *shouldn't* be used all the time (and my rebuttal):

- **What if you need to have a `` ` `` or `${}` in your code?**
    I cannot recall a single instance in my nearly three years of coding in NodeJs that I've had to use these in my strings. I doubt I'll come across more than a few such cases in my entire lifetime. And if I have to, I can always escape them [#BackslashFTW](https://www.youtube.com/watch?v=Ioh-oU1zn0w).
- **My keyboard has an European layout and using those backticks is a pain in the buttocks!**
    I can sympathize with this. If your tools go against you, there is little you can do. But in my personal case (and the case of my team), we don't have this issue. Hence, another point that I need not take into consideration (though you definitely should if you have such a keyboard).

So it doesn't look like there is much compelling research around why I *shouldn't* be using those oh-so-handy template literals. How about you guys find some for me and let me know?

And if you think I got anything else in this post wrong, let me know about that too.
