---
layout: post
title: "Semico what?"
excerpt: "A passing glance at semi-colons in Javascript."
modified: 2016-12-14T14:17:25-04:00
categories: code-craftsmanship
tags: [javascript, nodejs, coding-style]
comments: true
share: true
---

Recently, at my workplace, a colleague felt a [rustle up his jimmies](http://knowyourmeme.com/memes/that-really-rustled-my-jimmies) due to our lack of semicolon usage in our NodeJs repos. The number of projects without semicolons were in the tens and there really wasn't a good way to introduce semicolons in them even had the team agreed to do so. But that got me thinking.

> Do we really need semi-colons?

Why have them or not have them? What are the arguments for and against the issue? What do people argue with when they want to defend their position? So I started doing some light research.

> And yes, this was light research. I don't care enough about the issues to have spent more than an hour on it. Do your own research if you care so much.

## Stuff I Read
I read some stuff while researching this but these are the ones that I think you should start with. You *should* do your own research as well: don't take my word for it. Branch out from these links and find your own persepective on the matter. If you write a more comprehensive post on the issue, hit me up coz I'd like to read that.

- [An Open Letter to JavaScript Leaders Regarding Semicolons](http://blog.izs.me/post/2353458699/an-open-letter-to-javascript-leaders-regarding)
- [How this works...](http://blog.izs.me/post/3393190720/how-this-works)
- [JavaScript and semicolon](http://stackoverflow.com/questions/33644285/javascript-and-semicolon)
- [Why no semicolons?](https://github.com/expressjs/body-parser/issues/99)
- [Poll: Semicolons in JavaScript](https://news.ycombinator.com/item?id=1547647)

## This is What People Seem to be Saying (heavily paraphrased)

### Semico-yes:
- Not using it introduces weird bugs.
- My linter says so.
- Douglas Crockford says so.
- It's what I've always done.
- It's the standard!

### Semico-no:
- It leads to less code.
- It reminds me of Ruby,Python,[Brainfuck](http://www.muppetlabs.com/~breadbox/bf/).
- Isaac Schlueter says so.
- It's what I've always done.

> Right.

## My Unsolicited Thoughts on the Matter
- Using or not using semicolons in a language shouldn't be because you [miss using them/not using them in a different language](https://news.ycombinator.com/item?id=1547930).
- With the way things are today, our interpreters and engines being smarter than ever, a lot of things come down to a matter of preference, semicolon being one of them.
- That said certain things make sense about having semi-colons:
  - They *explicitly* terminate a statement. Anything that's explicit requires less of your mind-power to grok. You don't have to check the next line to see if a function call was terminated or chained.
  - -crickets chirping-

## Final Thoughts
I like semicolons for their explicitness. But at the end of the day, either style works. I use semi-colons in my personal code and don't in my professional. And I don't find it that hard to switch between the styles (especially since I offload any checking to my linters). So, use what you want, and if you can't because of whatever reasons, don't make a big deal out of it. It matters very little.

## Notes:
- http://blog.izs.me is this guy: [Isaac Schlueter](https://github.com/isaacs)
- If you want to understand statement termination in javascript really well, [check this out](http://inimino.org/~inimino/blog/javascript_semicolons).
