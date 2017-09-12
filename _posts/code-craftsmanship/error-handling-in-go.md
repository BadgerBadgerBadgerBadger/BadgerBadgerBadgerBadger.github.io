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

# Dealing With Errors in Go Projects

> Note: This post covers not just the technical aspects of error handling but also some good practices that have helped me debug errors in production systems.

I come from a primarily NodeJS background and my coding habits have been heavily influenced by the paradigms most used in Node. I started coding in Go quite recently and I immediately fell in love with the language.

Let's talk about what caused me a lot of headache and frustration: errors. I didn't know what to do with the darned things. The idiomatic way in Go is to return errors as a second value and that's what pretty much all libraries do. There is a panic-recover mechanism but that seems to be mostly used to recover from things like illegal memory access or nil pointer dereference (these shouldn't happen if you code properly).

I was looking down this barrel of handling and logging errors on the spot or returning them up the chain of function calls. Neither sounded like a good idea. Why? Here's why:

**If you handle errors on the spot and log them** you lose context of the chain of functions that led to this state. You can log variables local to the function, but you have no idea what path your program took to finally call this function and face this error.

**If you return errors up the chain** and then handle them at the topmost level, you end up with almost the same thing. You have the error and you know which entry point of your function eventually led to that error, but now you've lost context of the function where the error originated.

Neither sounds like a good idea, does it? Especially not at 4 in the morning when your boss calls you up to debug some obscure bug made more obscure by insufficient information. You need to know where the error originated, which entry point led to this and what path it followed. And of course, the details of the error itself.

To work around these issues, I had this idea of annotating the error message at each stage. So say I have the following extremely contrived code:

```go
package contrived

import(
    "thirdparty/lib"
    "errors"
)

func HandleRequest(req Request, res Response) {
    result, err := ControllerCall(req.Id)
    if err != nil {
        log(err.Error())
        res.BadResponse("...")
        return
    }
    
    res.GoodResponse("...")
}

func ControllerCall(id int) (*string, error) {
    result, err := DbCall(id)
    if err != nil {
        return nil, errors.New("controller failed to blah blah blah "+err.Error())
    }
    
    return result, nil
}

func DbCall(id int) (*string, error) {
    result, err := lib.DoSomething(id)
    if err != nil {
        return nil, errors.New("db call failed for id: "+string+" "+err.Error())
    }
    
    return result, nil
}
```

By concatenating the errors up the chain, I retain some context at each step. And at the top when I log, I can see the calls that led to the error. This is better than not having much context but is extremely tedious in practice.

Then I came across [this excellent blog post](https://dave.cheney.net/2016/06/12/stack-traces-and-the-errors-package) by Dave Cheney. And once you've read through it (seriously, read through it before continuing), you'll realize that he's implemented the same idea I had but done it a gazillion times better. So I started perusing more things he'd written. The presentation includes most of the content of the preceeding articles.

- [Constant Errors](https://dave.cheney.net/2016/04/07/constant-errors)
- [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Inspecting Errors](https://dave.cheney.net/2014/12/24/inspecting-errors)
- [Dave Cheney's errors package](https://godoc.org/github.com/pkg/errors)
- Dave Cheney's presentation on Error Handling [youtube](https://www.youtube.com/watch?v=lsBF58Q-DnY) [pdf](https://dave.cheney.net/paste/gocon-spring-2016.pdf)

After reading through all that and tinkering around for a while, I came up with a set of few sensible guidelines for handling errors. Some of them are taken directly from Dave Cheney's blog, some I added based on my own experiences.

## Let's Begin

- **Always** check for errors if the function returns one. I know that sounds kinda obvious but you'd be surprised at how much code I've seen that don't do this.
- Create custom error types for the errors you _know_ you are gonna face (bad input, duplicate request, etc.). Return these, wrapped by the `errors` package, when you face them.
- When dealing with errors returned from a library, wrap it with `Wrap` or `Wrapf` and return.
- When dealing with errors returned from your own code, if:
  - it requires no further context, just return it.
  - if it requires some extra context (perhaps data points captured within that function), use `Wrap` or `Wrapf`.
- Handle errors without further propagation at the entry point of your program (commands in a cli or handlers in a web server). Unwrap it with `Cause` and test it:
  -  If it is of the custom type you have defined for your project, handle accordingly (you might wanna log these at level Error or Warning since these were expected). If it's an `/api` endpoint, you might wanna have a contract with the client on what codes to return and act accordingly (you might even end up returing a generic error message). If it is an internal endpoint, you could return an appropriate status code and message while logging the same.
  -  If it is of an unknown type, you might wanna log a fatal. This is an error that should never have happened. After that it's up to you.

The above make my life much less tedious. I still have to do `error != nil` everywhere but I think it's a good thing. Unlike exception propagatin, this actually make me think at each step whether I wanna let the error propagate on its own or add more context.

## The Extra Mile

If you want to sink your teeth into Go's philosophy around errors, the following posts provide good grounding:
- [Errors are values](https://blog.golang.org/errors-are-values)
- Dave Cheney's presentation on Error Handling [youtube](https://www.youtube.com/watch?v=lsBF58Q-DnY) [pdf](https://dave.cheney.net/paste/gocon-spring-2016.pdf)
- [Dave Cheney's errors package](https://godoc.org/github.com/pkg/errors)
- [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Inspecting Errors](https://dave.cheney.net/2014/12/24/inspecting-errors)
- [Constant Errors](https://dave.cheney.net/2016/04/07/constant-errors)
