---
layout: post
title: Cookiefall Devlog#0
excerpt: "My first proper forray into game development using Unity."
modified: 2023-06-22T00:00:00-00:00
categories: game-programming
tags: [unity, tetris, germ-busters]
comments: true
share: true
---

Several things happened of late lead to me starting this project. I finally got into mobile gaming after having looked down on it for years, what with me being a "core" gamer, whatever that means. And to my plesant surprise, although the mobile games are filled with rampant monetization, the games themselves are actually quite fun. 

At about the same time I found out from my partner that one of her favorite games as a child was [Dr Kawashima's Brain Training](https://en.wikipedia.org/wiki/Brain_Age), a collection of minigames originally for the Nintendo DS, currently on the Switch. And the particular minigame she enjoyed was Germ Busters a game similar to [Dr Mario](https://en.wikipedia.org/wiki/Dr._Mario), something I had myself played as a child.

I had also started rewatching a lot of [Sebastian Lague's](https://www.youtube.com/c/SebastianLague) excellent series on coding adventures. All of this together lit a fire under my backside and made me want to seriously give game development a try.

The idea was to recreate Dr Mario, but for a mobile platform, and with legally distinct branding. I am still iffy about whether this would bring down Nintendo's wrath upon me. Is the idea of matching block colors something Nintendo can copyright. I feel like if there's one company that might try, it would be them. However, that's a problem for later. First I need to be able to actually build a game.

I started following [a Tetris making tutorial on youtube](https://www.youtube.com/watch?v=ODLzYI4d-J8&ab_channel=Zigurous) as the closest analogue to this type of game (which I'll call a Block Puzzler in the context of these articles). While several elements are different (only one shape as opposed to the 8 shapes(?) in Tetris, blocks clearing matched on color instead of lines clearing), it gives me a decent kicking off point.

I started off by defining a tilemap and using the same background as the tetris tutorial. However, I have immediately hit a roadblock.
---
layout: post
title: Cookiefall Devlog#0
excerpt: "My first proper forray into game development using Unity."
modified: 2023-06-22T00:00:00-00:00
categories: game-programming
tags: [unity, tetris, germ-busters]
comments: true
share: true
---

Several things happened of late lead to me starting this project. I finally got into mobile gaming after having looked down on it for years, what with me being a "core" gamer, whatever that means. And to my plesant surprise, although the mobile games are filled with rampant monetization, the games themselves are actually quite fun. 

At about the same time I found out from my partner that one of her favorite games as a child was [Dr Kawashima's Brain Training](https://en.wikipedia.org/wiki/Brain_Age), a collection of minigames originally for the Nintendo DS, currently on the Switch. And the particular minigame she enjoyed was Germ Busters a game similar to [Dr Mario](https://en.wikipedia.org/wiki/Dr._Mario), something I had myself played as a child.

I had also started rewatching a lot of [Sebastian Lague's](https://www.youtube.com/c/SebastianLague) excellent series on coding adventures. All of this together lit a fire under my backside and made me want to seriously give game development a try.

The idea was to recreate Dr Mario, but for a mobile platform, and with legally distinct branding. I am still iffy about whether this would bring down Nintendo's wrath upon me. Is the idea of matching block colors something Nintendo can copyright. I feel like if there's one company that might try, it would be them. However, that's a problem for later. First I need to be able to actually build a game.

I started following [a Tetris making tutorial on youtube](https://www.youtube.com/watch?v=ODLzYI4d-J8&ab_channel=Zigurous) as the closest analogue to this type of game (which I'll call a Block Puzzler in the context of these articles). While several elements are different (only one shape as opposed to the 8 shapes(?) in Tetris, blocks clearing matched on color instead of lines clearing), it gives me a decent kicking off point.

I started off by defining a tilemap and using the same background as the tetris tutorial. However, I have immediately hit a roadblock.
<img width="795" alt="image" src="https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/47f4b3a8-0eaf-490a-9968-050f79d19e43">
While on my editor the grid looks fine, especially with the camera set to a 16:9 portrait aspect ratio, on my actual phone, the grid goes offscreen (note how the mobile view only has 8 columns instead of 10).
![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/b6afec31-f4f6-457b-9b4d-0ddf6e693105)

