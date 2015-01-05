---
title: Functional Reactive Programming in Ruby (Part 2)
layout: post
date: 2015-01-04 20:29
categories: frp ruby roguelikes
---

### Last Time
I talked about wanting to build a roguelike game. I talked a bit about why I wanted to use reactive programming to make the input/output handling of my game easier. I talked about how I enjoyed functional programming. Now I’ll try to tie functional and reactive programming together.

### FRP context: Designed for GUIs
While I wish I could say I just wanted to take these two styles and mash them together, the combination was actually derived as an elegant and easy way to handle GUI event loops.

Most GUI frameworks use things like callbacks to handle events … you will set up (for example) a button that should receive clicks, but you will pass it a separate function that runs when the user actually _clicks_ on the button. In non-functional languages where you share state, this means you have separate pieces of code that keep track of the same thing, but since those callbacks can happen at any time you have to worry about timing and it’s really difficult to reason about how everything will happen. 

FRP is a way to handle the fact that program input probably affects the entire program end-to-end but you don’t want to have to share data in a way where one component changes things around for everybody else.

### Functional Reactive Programming
The way Functional and Reactive programming combine is through the concept of **events** and **behaviours**[^1]

[^1]: Sometimes frameworks combine the two together into a single concept, signals. I don’t know enough to know what the ramifications and advantages and disadvantages of this reframing is.

**Behaviours** are functions that, for any given time since the program started running, return continuous values for any given time when the program is running. So for example, in a web browser, you could have a behaviour that tells you what the size of the browser window is at any time. There’s always a window size as long as the program is running.

**Events** are a way to handle things like clicks or keypresses which aren’t continuous but behave similarly to the above signal functions. If a program is running, and I need to mouse click, I can have a function that returns ‘false’ for all times when nobody was clicking, and returns ‘true’ for all times when the user was clicking.

#### Why is this useful?
Doesn’t it seem like a lot of work and weirdness to be caring about the times in which anything happens? Doesn’t it seem like your program could take so much time updating all of these values and not doing anything useful.
Well, the nice thing about the reactive pattern is that it’s inherently lazy … even if the signals are changing all the time, you only care about their values at the moments in which you need them.

That being said, if you *did* track the values at every feasible instant, you’d have a nice snapshot of whatever you were making at every moment it was running. This would be really handy for debugging!

### Functional programming around signals
The really neat things about these signals is that a lot of the basic functional programming operations can be made to work on these signal. in FRP, these operations[^2] will return streams that are modifications (often supersets or subsets of the original signal).
If the system provides you with a signal that informs you of every keyboard press in the program, you could have a signal that *filters* out any keypress that isn’t the few you are interested in. You could *merge* two signals together so that you get their combined values (and perhaps then produce an even more complicated filter)

[^2]: I’m not going to discuss these basic operations, if I wanted to be responsibly comprehensive I totally would though. Here’s a [link](http://learnyouahaskell.com/higher-order-functions) in case you’re interested

You could have a *map* function that always applies a function to a signal’s value and returns the derived value.

From a time point of view, you might have a *sampling* signal that only returns a value every so many seconds or microseconds. This will allow you to not clog your program with unnecessary updates while still keeping things up to date. 

A really neat one is *reducing* or *folding* over a stream … given an initial value and a function that takes that initial value and a value from a signal, you can produce a new value using the signal’s value and the initial value. When the next value comes down the signal, we take the derived value and the new signal value and produce something new which we pass into the next iteration, and so on, and so on!

Spitting output back to the user is not *technically* stateless, and functional programs often struggle to separate out these necessarily stageful parts of the program from the functional stateless core. The nice thing about signals is that you could have a callback here that renders some signal value to the screen, and this can be one of your only callbacks. In programming environment[^3] that are designed around FRP, this update callback might be hidden from the user so that the user doesn’t have to worry about it.

[^3]: Like Elm, the realization of FRP that I learned al this stuff from. Because it’s browser-based, the entire program consists of writing a particular kind of signal which the environment 

### Next time.
I’ll show you what this code looks like in Ruby so you can see how it works!