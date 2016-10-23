---
layout: post
title: First Foray into C++ with SFML
excerpt: "My first adventure trying build something in C++ using the great SFML framework."
modified: 2016-10-23T14:17:25-04:00
categories: game-programming
tags: [c++, sfml]
comments: true
share: true
---

Game-programming has always been one of my interests ever since I coded a simple game of ![Pong!](https://www.openprocessing.org/sketch/158668) back in college using ![Processing](https://processing.org/). But my first job ended up being in web-development and from then on that’s all I’ve been doing. It is easier to get a foot in through the door in web-development than it is in game-development (I don’t have a formal Computer Science education), and hence, two years into being a earning member of society, I’m still in web-development.

But my interest in game-development never went away and so recently I picked up C++ and the SFML library to try to see what I can learn and what I can make.

After figuring out how to set up ![SFML for XCode](http://www.sfml-dev.org/tutorials/2.4/start-osx.php), I got into the thick of things and started coding. It’s been a helluva 10 days and here are some of the things I think are worth noting down because, well, they gave me trouble. And it’s always worth remembering the things that give you trouble.

My goals with my first program were simple. Make a sprite that moves across the screen based on user input (Left, Right, Up, Down). If the user keeps holding the button, the sprite accelerates. If the user lets go of the button, the sprite keeps moving with whatever velocity it had achieved during acceleration. There is also a constant drag force acting in the opposite direction of the sprite’s movements that would eventually bring it to a stop when no longer acted upon.

> Simple enough.

# Headers and Header Guards?

One of the first things I had to conceptually decide on was how to structure my header files. Header files and the whole concept of forward declarations doesn’t exist in Java and Javascript (the two languages I’ve used in the past) and so working with header files was a completely new experience for me, especially when it came to project organization. I faced a binary choice here (if there are more ways of doing this, someone do let me know): 

Do I keep them along with the source file, each pair being stored as a unit? Or do I keep all the headers in one place?

![Code::Blocks](http://www.codeblocks.org/) organizes it’s projects by separating out its files into `source` and `header` trees, and the SFML library itself has headers stored in one folder under a hierarchical structure. Although coding in modern IDEs is easier than ever (you move a file and the IDE **updates all the hundreds of references** for you!), I still wanted all of my headers to be in one place and referable from one root location. So I put them all under the `header` directory in the root of my project.

Another thing that tripped me up were the ![header guards](http://www.learncpp.com/cpp-tutorial/1-10a-header-guards/). I understood what they were and I understood how to use them, but I let the IDE do the job of creating the guards when I created new header files (the guards were named after the path of the file inside the `src` folder; example: `MOVE_COMPONENT_GRAPHIC_H`) with the result that after I’d moved and renamed some files, the header guards did not update (the IDE is not that smart), so I ended up with two header files with the same guard.

> And what happened when I included both in my main.cpp?

The second one did not import and I couldn’t figure out why my struct wasn’t usable in the current scope. It took me a good hour to realize that there was nothing wrong with the IDE or the compiler. I had simply not taken the header guards into account.

I still let the IDE generate the guards for me (no point in having an IDE if it doesn’t do the heavy-lifting), but I always double-check to ensure that no other files have the same guard. Doing a quick text-search across the project does the trick.

# nan, nan, nan, nan, nan…

I didn’t realize, initially, that SFML had its own Vector classes (I didn’t RTFM, just jumped into code; Kids, don’t do this. Always ***RTFM!***), and so went ahead and wrote one of my own. A 2D vector class that had x and y members and the usual methods for vector computations, including a method to find the vector’s magnitude and one to normalize it. ![Now here is how you normalize a vector.](http://www.fundza.com/vectors/normalize/) As you can see, if the magnitude is zero, you’re in for some trouble.

Now in Java a division by zero is thrown as an ![Exception](http://www.studytonight.com/java/exception-handling.php). In NodeJS, a division by zero yields ![Infinity](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Infinity), but for C++, a division by zero yields ![nan](http://www.cplusplus.com/reference/cmath/nan-function/) of the appropriate type (yes, a nan for a double is not the same type as a nan for a float, just like 0 and 0.0 are different types). I didn’t know this would happen, and I’d forgotten to account for vectors of 0 magnitude.

> So, what happened?

Nothing. Literally nothing happened. As in no computation would work because I as computing with nan all over the place and nan computed with any number gives you *nan*. Took me some printing of the values to figure this one out, though in hindsight this should have been laughably obvious.

# The Drag

Another idiotic thing I did was in writing the drag function. Here’s the first version of that.

```cpp
void scionofbytes::apply_drag(scionofbytes::Mover& mover) {

    scionofbytes::Vec2D drag = mover.velocity.scale(-1);
    opposite_velocity.normalize();
    opposite_velocity.scale(scionofbytes::DRAG_COEFFICIENT);

    mover.velocity.subtract(drag);
};
```

Anyone see what I did wrong here? Anyone?

I’ll wait.

…

…

…

Okay, if you haven’t figured it out already, I subtracted the drag vector when I should have been adding it.

See, I’d already computed the drag force as a vector in the opposite direction to the velocity vector. And by subtracting this vector instead of adding it to velocity, I’d just increased the velocity.

It’s like subtracting a negative number: you actually end up adding move value. See?

Anyway, those were just some of the things I struggled with in my early delving of C++ and SFML. And all of them weren’t even related to programming. I’ll keep posting my adventures as they take place.

Cheers!
