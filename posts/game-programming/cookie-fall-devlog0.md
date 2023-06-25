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

<img width="252" src="https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/b6afec31-f4f6-457b-9b4d-0ddf6e693105" />

I tried several things to make this dynamically change with the size of the screen with code as terrible as this:
```csharp
// We need to adjust the grid to fit within the world as per the screen size.
// So first we get the world width, then we divide it by the number of cells
// we want to fit in the world to get the cell width.
var worldWidth = (int)(Camera.main.orthographicSize * 2 * Screen.width / Screen.height);
var cellWidth = (int)(worldWidth / boardSize.x);

// Now we set the cell size of the grid to the cell width.
// _grid.cellSize = new Vector3(cellWidth, cellWidth, 0);
_grid.transform.localScale = new Vector3(cellWidth, cellWidth, 0);
_gridSprite.transform.localScale = new Vector3(cellWidth, cellWidth, 0);
_tilemap.transform.localScale = new Vector3(cellWidth, cellWidth, 0);

var difference = 1 - cellWidth;
var yOffset = (boardSize.y / 2f) * difference;

var gridPos = _grid.transform.position;
_grid.transform.position = new Vector3(gridPos.x, gridPos.y - yOffset, 0);

var gridSpritePos = _gridSprite.transform.position;
_gridSprite.transform.position = new Vector3(gridSpritePos.x, gridSpritePos.y - yOffset, 0);
```
However, due to the imprecision of floating point calculations, getting sensible even numbers was nearly impossible. Even though I could get the cells to size dynamically, repositioning everything with pixel perfect accuracy proved difficult.

<img width="252" alt="image" src="https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/87b38651-53d9-4698-b919-d33865d37934"><br>
Things never aligned quite right.

Finally, after a lot of research, and looking into how other people solved this problem (which is something I should have done from the start), I realised I needed to look at the problem differently. 

Instead of having the board fill up the entire screen, which would also not leave any space for other UI elements (a problem I had even considered),the grid can take up a part of the space with enough buffer at the sides to account for any but the wackiest of screen ratios.

With the camera size increased to 11, here is how it now looks with 16:9 and 18:9 aspect ratios.

![sc169](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/bdeb7a2e-04a7-4bc7-8bbd-87e91f74698b)![sc189](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/00d85846-e6c5-4cda-8319-0c6c70644c4c)

And with that one hurdle looked like it would be no hurdle at all.

![image](https://github.com/BadgerBadgerBadgerBadger/BadgerBadgerBadgerBadger.github.io/assets/5138570/0748fd1c-28e5-456e-b6e5-21551c20a6b8)

