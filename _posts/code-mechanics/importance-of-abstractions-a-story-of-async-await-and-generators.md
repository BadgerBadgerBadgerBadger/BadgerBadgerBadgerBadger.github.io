---
layout: post
title: "Importance of Abstractions - A Story of Async/Await and Generators"
excerpt: "Understanding why it is important to fully grok the underlying abstraction of a system."
modified: 2017-14-22T14:17:25-04:00
categories: code-mechanics
tags: [javascript, nodejs, coding-mechanics]
comments: true
share: true
---

> Disclaimer: You'll need to understand how [JS generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) work, a little knowlegde about the [Async/Await](https://blog.risingstack.com/async-await-node-js-7-nightly/) mechanism and a good understanding of [Promises](https://gist.github.com/domenic/3889970) in general to get much out of this article.

**TLDR**; *Understanding abstractions is important. Promises are the underlying abstraction of async/await, generator/coroutine and libraries like [Bluebird](http://bluebirdjs.com/docs/getting-started.html) and [Q](https://github.com/kriskowal/q).*

I was helping a co-worker debug an API at work and we came across a rather strange bug. If you have ever used [async/await](https://ponyfoo.com/articles/understanding-javascript-async-await) or the [generator/coroutine](http://tobyho.com/2015/12/27/promise-based-coroutines-nodejs/) functions in Javascript (which themselves are a simulated version of async/await), you have seen code like the following:

```javascript
const co = require(`bluebird`).coroutine

/**
 @param {Number} gregsNumber
 @returns {Promise}
 */
function callGreg(gregsNumber) {
    return co(function* () {
    
        const reponse = yield callNumber(gregsNumber)
    
        if (!reponse.error) {
            return reponse.reply
        }
        
        throw new PhoneError(response.error)
    })()
}
```

Leaving aside the fact that this is a very contrived example and some of you may be appaled at the way I'm wrapping that error, this pattern should be familiar.

I've defined a function that returns a Promise. It does so by wrapping a generator inside Bluebird's coroutine function allowing us to call functions like `callNumber` in a synchronous-looking fashion without actually blocking the [libuv](https://github.com/libuv/libuv) [event loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/). This is a boon to anyone who has been programming for a long time in JS and has had to deal with [callback hell](http://callbackhell.com/) and later [promise hell](https://medium.com/@pyrolistical/how-to-get-out-of-promise-hell-8c20e0ab0513) (a lesser hell, for sure, but I'd rather not have any kind of hell).

The generator/coroutine mechanism above abstracts away the underlying Promise (which in itself works by abstracting away callbacks), and lets us reason about our code in a synchronous fashion without having to worry about the details.

But it is important to realize that the underlying mechanism is still Promises. The `callGreg` function returns a Promise which can then be used by the caller function in multiple ways:

```javascript
const co = require(`bluebird`).coroutine

function iUsePromises(gregsNumber) {
    return callGreg(gregsNumber)
        .then(gregsReply => /* do something */)
        .catch(error => /* handle it */)
}

function iUseGeneratorAndCoroutine(gregsNumber) {
    return co(function* () {
        
        try {
            const gregsReply = yield callGreg(gregsNumber)
            // do something
        } catch (e) {
            // handle it
        }
    })()
}

async function iUseAsyncAndAwait(gregsNumber) {
   try {
            const gregsReply = await callGreg(gregsNumber)
            // do something
        } catch (e) {
            // handle it
        }
}
```

All three functions can use `callGreg` though they work in different ways. That is because they have been been promised (😉) a Promise. It doesn't matter whether we implement `callGreg` with Bluebird Promises, Q promises, a coroutine or even asn an async function. The contract of a Promise has been fulfilled.

Let's get back to my story. As I've mentioned above, we had a problem. Our code was not behaving in the synchronous fashion we had expected and was jumping around and executing out of order. Finally, my sharp-eyed co-worker found the problem.

This line of code was the culprit:

```javascript
yield renamedCozHaha()
```

You won't notice anything amiss till you see the definition for `obfuscatedForSecurityHahah`:

```javascript
function* obfuscatedForSecurityHahah() {
    // it does something
}
```

Here's what's going on.

When you call a generator function from another generator function, you can iterate through the called function and get the resultant value all in one expression. But this only happens when you use the [`yield*`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*)(`yield*` itself is an operator that delegates to a JS iterable) operator instead of the regular `yield` and the problematic code does not have the `*`.

You could write that off as a typo and rightly so; I've done much worse and seen much worse. But this would have been much less likely to happen if the contract of Promises was respected in the first palce. 

Early on, when the generator/coroutine pattern was availble for use from Node4 onwards, we had embraced it to make our mostly Promise-based code easier to reason about. We had agreed on the `co(function*() {})` pattern for handling asynchronous control flow. 

But in embracing the syntax and specific implementation of that pattern, we had lost sight of the *why*. *Why* did we use that pattern? *What* were we trying to abstract? 

The point of using the generator/coroutine mechanism was not because we explicitly wanted to use generators or coroutines. It was so that we could reason about our code in a synchronous fashion while using Promises as the underlying abstraction.

A generator function by itself is not compatible with Promises. It cannot be called with a single `yield` expression from other generators, it cannot be called with an `await` expression from an async function and it is not [thenable](https://promisesaplus.com/#terminology). It is completely outside the realm of the Promise abstraction. The JS Generator and its details is not the point. The point is the abstraction provided by Promises. *That* is what we're ultimately trying to work with.

We haven't started using async/await in our codebase yet (waiting for Node 8 to go into LTS), but when we do, integrating it with existing code is going to be trivial. All functions are already compatibale (since they all return Promises) and can be called with the `await` keyword from any new async functions written.

# Conclusion

The generator/coroutine mechanism is not useful for its own sake but because it provides a synchronous looking syntax on top of the actual abstraction provided by Promises. Replace that with async/await and your code will keep working without a hitch.
