---
title: Catamorphism for dum dums (Functional Programming in Scala, Chapter 3)
layout: post
date: 2016-01-10 18:43

---
[**Caveat:** This is a very technical post and I feel like I should do the due diligence of having a glossary that is easy to reference (and doesn't require me to re-explain everything on every post. But I'm kind of intimidated by this thought, and I need to figure out how to pin it in the software I'm using to host a blog. Please forgive me :D]

Hi. I want to be fluent in Haskell and type theory and programming language theory in general. I am also not very smart.

I have a housemate who is fluent in Haskell. Every day I would come home and ask him what a Monad was. Usually within 30 seconds the words Monoid, Functor, and f-map would pop up. Every day I would try to understand each point so that I could know what a monad was. Every day I would take slightly less time to re-learn each concept, and the understanding would last slightly longer each time before I forgot or re-confused myself[^1]

--

I recently slammed enough of my head against a Clojurescript hobby project that I got an minimum viable product in terms of functionality. At work I regularly work in PHP and Javascript. That month was filled with a lot of silly type errors (all of these languages are dynamically typed and don't check for variable type consistency until runtime) and I decided it was time I wrote a project in Haskell once and for all[^2]

--

I know how higher order functions work. I know what immutable data structures are, and why tail recursion is nice. I know what `map`, `filter`, and `reduce` do.
But what I don't know is HOW you structure a functional programming language.

I know that it's often nice to write little functions, play with them in REPLs to make sure they work, then write the software at a higher layer. I realize that functions are less difficult to organize than classes (since classes are a mess of behaviour and data, and functional programming langauges emphasis data structure outputs over states.

But I still feel there is some kind of _structure_ to writing a complex functional programming project than people let on. Like even if nobody thinks of it as such, there must be some kind of _design pattern_ or something. Right? **right??**

--

I recently picked up a copy of [Functional Programming in Scala](https://www.manning.com/books/functional-programming-in-scala) that was getting a lot of attention recently. I think Scala is an atriociously syntaxed language[^3] even if it's probably as capable an ML-inspired language as anything out there. But what this book seems to promise is to teach you _via Scala_ how to think in terms of writing functional programming projects.

I spent my holidays going over the first three chapters, and I'd like to write a little about the book as I've so far experienced it.

--

So the first chapter talks about the motivation, and everything sounds good given my general understanding of why functional programming is teh awesome.

The second chapter introduces you to Scala and its syntax. I am glad I know other functional languages like Standard ML and a few Lisps, because I do think Scala is a little clunky sometimes and it would suck to learn the concrete particulars for things that aren't actually all that necessary in themselves. But unfortunately that's going to happen if you aren't experienced enough yet to juggle the various levels of abstractions. See how I'm such a Lisp lover and how that's made me suspicious of the feasibility of types and threatened by the usual loss of homoiconicity and macros :)

--

So this book asks you to do the exercises, which is usually the part of any book which stumps me because I get stuck and then I have to look on the internet for answers and then I feel dumb and then I stop. Do I have a fallacy of assuming one has to work through everything in order to understand it all. Is there room for forgiveness? I don't know. But this is the biggest reason why I still haven't finished the canonical [SICP](https://mitpress.mit.edu/sicp/), because unless I search the internet for answers I trust I have no idea whether I actually understood the subject or not.

I promised myself I would do the exercises in this book, because unlike most books this book doesn't attack the subject from a programming language theory or math framework (two things I never really understand and always feel stupid about)

--

The third chapter was where I started struggling. The chapter asked me to implement that most basic of data structures, a linked list. It asked me to implement functions like map, head, reverse, append, etc. It then introduced higher order functions, whatever, I know those, right?

It asked me to implement `foldLeft` and `foldRight`. The TL;DR that I probably can't explain well is that the former is easier to understand and is easy to optimize for performance via tail-calls, whereas the latter is more general (foldLeft has some limitations if you're trying to use it for certain things) and the canonical implementation is not very performant because you can't do tail-call optimization on it. Anyway, that took some time, and my housemate had to remind me that graphing the functional calls for an imagined invocation would be a good idea, and it was :D

I was then asked to implement foldLeft _in terms of_ foldRight and vice versa. This took me almost a week to figure out[^4]. But thanks to the call graph my housemate mentioned I figured it out. Hurray! This was the hardest exercise I've had to do thus far.

The rest of the exercises were implementing the standard functions again in terms of fold, and then implementing a Tree structure and using a fold function to implement all the basic functions you want for a tree.

--

Something clicked in my head as I was working on the tree stuff (which was very easy in comparison now that I grokked the concepts).

Like I knew that `map` and `filter` and `fold` are important functions, and really useful. And I sort of knew of the left/right distinctions for fold, and I sort of knew why although I'm not experienced enough to ned to implement something where it matters.

But what is super powerful now is knowing that `foldl`/`foldr` doesn't _really_ matter if I check that they are implemented properly. Like, I can use the one that best expresses the problem I want to solve now since ultimately they can resolve to the same implementation.

Also, given that I have an understanding of the usual library of functions one has for basic functions like lists and trees and hashes and such, there seems to be this pattern that if you have to iterate over an entire list, you can generally implement it in terms of `fold`. And this suggests that `fold` is a very important function, more important than I know.

I talked about this with a Haskeller and they immediately chimed up and said **catamorphism!** And that's a word I've heard casually in haskell spaces before, but it was nice to understand it via a back door, because the usual math approaches to explain such things to me just don't make sense given my lack of mathemathical discipline.

--

Is this why people attempt proofs? Did I gain these insights because I struggled with the material until I had enough insight to do the carefully curated assignments? What do you do when the proofs are very hard? Has math pedagogy worked around the areas where the reasons things work are not intuitive?

--

An interesting Scala idiom is the `flatMap` function, which is a map-then-flatten function. So for example in a list, you might supply a map function with a function that returns a list of items. So now you'll have a list of a list of items, one list for every element in the original list.

A `flatMap` will, given a function that returns a list of things for every element in the original list, will return a single list, i.e. flattening the inside lists.

I don't know why this is useful, but as I talked to my Haskell friends a lot of them have said that this is the function that will be critical to eventually understanding what Monads are.

This book is showing me `flatMap` functions for every data structure it throws at me, making me wonder if there's a standard set of functions that you would expect any data structure to have. And do these form the algebra (i.e. set of rule sand operations for the data structure) that allow me to think of all common data structures in an almost-unified way?

Maybe this is what the 'algebra' in an 'algebraic data type' means, which I believe is what a lot of these ML-based languages work with. But perhaps I'm reading too much into this.

--

Anyway, the point is that I really like this book because it's so far living up to the promise of making me have a _thought process_ behind functional programming and structuring a progam, not just the "map/reduce/filter is awesome!" that most intro books seem to stop with.

[^1]: My housemate is the very soul of patience.

[^2]: Admittedly I know Standard ML and the basic parts of OCaml and I've faked an undergraduate class on programming language theory, so other than being incredibly slow and addled once a thing involves math, I have to admit I'm not completely new at this.

[^3]: Please remember that I am a dum dum and this is based mostly on subjectivity and an emotional attachment to Lispy languages.

[^4]: hahahaha \*sweats\*
