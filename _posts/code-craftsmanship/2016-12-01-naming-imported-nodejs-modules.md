---
layout: post
title: Naming Imported NodeJS Modules
excerpt: "How I name the modules I import into other modules, in NodeJS."
modified: 2016-10-26T14:17:25-04:00
categories: code-craftsmanship
tags: [c++, sfml]
comments: true
share: true
---

*Disclaimer: This isn't a tutorial, guide, or holy writ. This is just how I do things and I'd like to share why it makes sense to me. If it resonates with you, follow it. If it doesn't, follow whatever makes sense to you and your team.*

Styleguides are something I take very sseriously. Give me a programmer who can write clean code with strict adherence to the style, who understands and respects the tenets of readability and judicious use of whitespace, and I'll always take them over people who can write clever algorithms.

I follow a very specific naming convention for when I import modules in NodeJS. They differ for on-site modules and vendor modules but for each, the rules are [mostly] consistent.

Here's what I do and why I do it.

Project Modules
---------------
These are the modules you write yourself as part of your own project. I won't go into how you should structure your source files or name them: that's beyond the scope of this article. But let's talk about how you can name them once imported.

Previously, I used to be of the *name everything in camelCase* camp. I literally used to name everything in camelCase without any thought to context or suitability. Why? Because everyone else did it, so why not?

After I joined the company I'm working at (as of this writing), I was introduced to a new styleguide, one of the rules of which was: **Use uppercase variable names for imported modules.** I went along with it coz I did not see a reason not to. It's actually one of the conventions picked up from the [Hapi styleguide](http://hapijs.com/styleguide).

Now the Hapi styleguide honestly sucks. Not because any of the conventions are wrong: I'm not sure if conventions, by their very definition, *can* be wrong ([correct me if I'm wrong on this point](mailto:shuvophoenix@gmail.com)). Conventions just *are*.

I have a whole separate post for why I think the Hapi styelguide sucks [here]().

After thinking about it for a while I realized how PascalCased names can be advantageous in certain situations.

In Java, with its concepts of classes, interfaces etc. A class or interface would be PascalCased to indicate that it's not an instance but a class. Its funtions would be static without an object context. Instantiated objects would be camelCased and have non-static methods.

Javascript, though object-oriented, does not have classes or interfaces. It's inheritance is based on [prototypes](http://javascriptissexy.com/javascript-prototype-in-plain-detailed-language/). Moreover the *object* in Javascript can also be used (and is extensively used) as a dictionary type in other languages. It's quite a versatile little thing.

Now, what's the advantage of naming modules required from within your own project in PascalCase? You can immediately differentiate between those that are instances vs. those that are just plain objects.

When a module is imported as such:
```javascript
const MyModule = require('path/to/my_module')
```
Most likely it will be composed of functions that should be used as-is without a `this` context attached to them. Those functions are just functions and not methods i.e. they don't have any object they are bound to. You can freely pass these around without any issues.

On the other hand, when you have a module that exposes an object instantiated from a  *class* (I know I said Javascript doesn't have classes, and it doesn't; read [this](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) for clarity), or a [constructor function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/constructor), you expose it as such.

So say I have an instance of an AWS Lambda service. I'd expose it as so:
```javascript
const AWS = require('aws-sdk');
const lambda = new AWS.Lambda({apiVersion: '2015-03-31'});

module.exports = lambda;
```

And I import it:
```javascript
const lambda = require('path/to/lambda.js')
```

Vendors like AWS also use this convention where a module has functions that are free of a `this` context, are in PascalCase to indicate that they are *class like*, and the instantiated object itself is in camelCase.

It's a subtle difference and isn't the *best* convention (if you find the *best* one, do let me know), but it makes sense and it works for me.

To summarize:
- If I'm exposing a module with functions not bound to any objects, I think of that module like a class in Java where all the functions are static ones. These are always PascalCased.
- If I'm exposing a module which is already instantiated- hence its methods are bound to the object- I think of these like objects in Java. These are camelCased.

It's that simple, really.

Vendor Modules
--------------
These are the third-party libraries you get from the npm registry (pretty-much 99% of the time), or directly via a [github remote url](http://stackoverflow.com/a/17509764/2584375).

My policy with vendor modules is this: I try to name them the same way the vendor has in their example code (github/npm/website/blog).

> Why?

It has to do with one of my biggest policies as a programmer: **Write readable code.**

> How does following the vendor's own convention for their library's usage help in code readability?

Here's how:
- Using libraries the way that the owner/maintainer uses it means that anyone looking at it for the first time will know exactly what it is. The same usage on the owner's site is also the way it's used in my project.

- Since the usage is what's shown on the library owner's GitHub/Npm page, that's most likely how other people have used it too. Stackoverflow posts, tutorials, blog posts, examples: reading other people's code/blogs about the library becomes easier if you see a library being used in the same way.

This seems like a small thing, but when you're reading code a certain number of weeks down the line, or debugging stuff by googling it, having vendor libraries named in the same way everywhere really helps.

There are exceptions to every rule, of course, and one of mine is the [`coroutine`](http://bluebirdjs.com/docs/api/promise.coroutine.html) function from [Bluebird](http://bluebirdjs.com/).

I always import it as `co`.

> Why?

- I make use of the couroutine pattern extensively (at least once in every asynchronous function) in my code and using `co` instead of `Promise.coroutine` simplifies life.

- Using `co` this way is not actually my own invention either. [TJ Holowaychuk](https://github.com/tj)'s [`co` library](https://github.com/tj/co) has the same pattern. I use Bluebird's implementation because it's faster (sorry TJ &#x1f605;) but TJ's code is familiar enough that I think others (and myself) will understand what the intent of the function is.

So a little bending of the rules when necessary.
