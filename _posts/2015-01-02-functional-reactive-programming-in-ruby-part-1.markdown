---
layout: post
title: Functional Reactive Programming in Ruby (Part 1)
categories: frp ruby roguelikes
---

A few months ago I decided to write a [Roguelike](http://en.wikipedia.org/wiki/Roguelike). As someone who has professionally developed Operating Systems and has an enthusiastic fascination for such 80s monstrosities as Common Lisp and Emacs.

<img alt="My (first) Roguelike" width="700px" src="{{ site.baseurl }}/assets/img/ykwya_1.png" />

### Ruby! 
I ended up choosing Ruby because it’s a language I’d love to work with professionally, I find it incredibly powerful and expressive and easy to understand (which I realize isn’t everyone’s experience, and I’ve definitely seen some pretty wide-spread bad practices.)

One thing I absolutely love about Ruby is how easy it is to write in a functional style and minimize the need for mutable state.

### Reactive Programming
I’m a backend/infrastructure person by trade, I usually focus on how input should be taken in and how it should be spit back out after processing. The little work I’ve done dealing with interactivity has always been my least favourite stuff because it’s always grasping at unknowns and basically shuttling events from the scary user to the interesting bits.[^1]

[^1]: I think empathy for the user and the whole concept of user experience and accessibility are awesome things! I’m just not great at them myself, and it’s not a thing I derive much passion from.

Reactive Programming has emerged as a pretty handy way for user (and other!) inputs to propagate into program logic. I’m sure it’s expressed better elsewhere (for example, [here](https://www.youtube.com/watch?v=4L3cYhfSUZs)) but basically if you have one component that keeps sending input or events to observers, the program logic can be written to only perform when new input has come in. Similarly, the UI’s output can be set to react to new game state by immediately rendering it. It’s pretty neat!

[The Observer Pattern](http://sourcemaking.com/design_patterns/observer) is how you get classes to act notify others that something has happened, and frameworks like [Rx](https://rx.codeplex.com/) framework can enable notified objects to immediately act upon notification without you needing to write the code which makes that happen.

### Functional Programming.
Functional Programming is a signifier of distinction for much of the programming community, and I’m a fan of it. As I was figuring out how I’d lay out my Rogue I started off using a bunch of objects and mixing, but as I got to the crunchy bits where I have to make the game world advance each turn I realized that I was constantly having to ruminate over which object ought to have been responsible for what logic, and this forced a lot of code just for dealing with how these various objects were made available and how interacted with each other.[^2]

[^2]: (There’s actually a whole stuff about functional programming that I’m not going to talk about, like how you can pass functions themselves into other functions so that you work at higher levels of abstraction, and how now your code is less about how you shove data around but can better express the high-level calculations you are performing. I suspect there’s a lot about my enthusiasm for Functional Programming that won’t make sense, but I can’t think of a quick explanation that isn’t a [textbook](http://www.manning.com/bjarnason/) or anything, sorry  ;_;)

Additionally, with all these different objects changing data all over the place, when things went wrong it was very hard for me to reason about how my code got into the incorrect state it did (especially when things went wrong over time and I didn’t want to write auditing code just to trace it.)

By keeping all of the game’s state together in one data collection, splitting all the logic into many manageable chunks was a lot easier then having logic and data spread out all over the place in tiny little interleaved pieces. I stored the entire game state in hash map, each of my functions would take it as an input and return a _new_ copy of the map with processed data for moving the player, attacking monsters, etc…

Game state wasn’t stored in countless objects that could change at any given time. I could, if I wanted, see the states of the world on every tick in one consolidated place. It was far more important for the logic of my code to be broken up.

I have to say though, Ruby’s syntax is beautifully expressive, but the language isn’t designed to have no state changes whatsoever. My code has this Scala vibe to it, which is kind of fun :/

Also, I have to contort my code to clone objects all the time, since Ruby doesn’t do that on assignment (and why would it, if you were changing state as you normally would?) This was a bit annoying, but I have only myself to blame :P

#### Next time
* I’ll talk about what happens when you combine Functional and Reactive programming
* I’ll talk about a really neat framework that does this really effectively for the browser :D
* I’ll talk about what I used instead, and what I had to do to make it work :P

***
