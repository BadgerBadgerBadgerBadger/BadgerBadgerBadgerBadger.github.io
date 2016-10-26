---
layout: post
title: Working with SFML::Textures — The White Box Problem
excerpt: "Working with SFML textures is pretty easy, but there are some things you'll still have to watch out for."
modified: 2016-10-26T14:17:25-04:00
categories: game-programming
tags: [c++, sfml]
comments: true
share: true
---

So I started using textures in my code, and it’s not been a fun time, mainly because I was missing some key understanding of how c++ keeps things in scope. Sadly I still don’t understand how what I did solved things, but I guess I’ll find out with more research.

Anyway, I’ll document what I did so that anyone trying this out can see exactly what I did wrong. Or rather which things I did wrong. I still don’t fully understand the root cause.

I first started out by defining a Ball class (I’m making a Pong! game), like so:

```cpp
class Ball {
    public:
        unsigned collider_size;
        scionofbytes::MovementComponent movement;
        scionofbytes::GraphicComponent graphic;

        Ball(u_int init_collider_size, std::string texture_path) {
            collider_size = init_collider_size;
            movement = scionofbytes::MovementComponent();
            graphic = scionofbytes::GraphicComponent(
                    (u_int) collider_size/2,
                    (u_int) collider_size/2,
                    texture_path
            );
        }
};
```

The Ball class has a collider size (which should probably be its own component), a movement component and a graphic component. The Ball constructor would receive a `texture_path` as a string and pass it on to the `GraphicComponent`'s constructor.

The GraphicComponent looks like this:

```cpp
class GraphicComponent {
    unsigned height;
    unsigned width;

    public:
        sf::Texture texture;
        sf::Sprite sprite;

        GraphicComponent() {}

        GraphicComponent(unsigned init_height, unsigned init_width, std::string texture_path) {
            width = init_width;
            height = init_height;

            texture.loadFromFile(texture_path);
            sprite.setTexture(texture);
        }
};
```

I use the `texture_path` to load the texture into the `sf::Texture` type, and then pass that on to the `sf::Sprite` (the sprite doesn’t store a copy of the texture, only a pointer to it; the texture must exist for as long as the sprite is using it).

Now the point about needing to keep the texture alive for as long as the sprite was using it is something I understood. What I couldn’t figure is why the texture kept getting destroyed once I’d instantiated the `Ball` class like so:

```cpp
scionofbytes::Ball ball = scionofbytes::Ball(50, "path_to_texture");
```

I’m creating the texture as part of the new ball object and as far as I could figure, it’s created as a local member of that object. So if the object is alive, the member is alive. Hence, passing the texture to the sprite shouldn’t be an issue. If the `ball` object is alive, so its `GraphicComponent` and so is the texture.

But there was still something going wrong and the texture was being lost before the system started drawing the sprite to the screen.

> And what happens when you draw a `sf::Sprite` to the screen whose texture is lost? You get a White Box (of death|frustration|sadness).

So how did I go about fixing this? I created a holder for the textures that would always be in scope and then passed the texture’s pointer to the sprite.

```cpp
scionofbytes::TextureManager textureManager;
textureManager.add_texture("ball", "path_to_texture");

scionofbytes::Ball ball = scionofbytes::Ball(50, textureManager.get_texture("ball"));
```

The `TextureManager` class just stores textures in a map of type `std::map<std::string, sf::Texture>` and the `get_texture` method returns a pointer to a texture by its key. This pointer is then passed on to the constructors which have been changed to accept pointers.

The `GrpahicComponent` class now has a `sf::Texture*` (pointer) type instead of a texture itself and passes on the same to the sprite (which was storing a pointer, anyway).

And it all works.

![KirbyBall](http://i.imgur.com/kx5lejh.png)
