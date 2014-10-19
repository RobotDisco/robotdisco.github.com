---
layout: post
title: "Starting out with Common Lisp"
categories: common-lisp
---

I apologize for writing a post that's most likely superfluous, but if I have Common Lisp on my brain I probably should talk about how I got started, in case it's useful.

### Motivations
I suppose first, _why_ am I so into Common Lisp. That's probably a series of posts for when I am more articulate about programming, but there is something incredibly powerful about a language that is so powerful despite having such a simple syntax. Common Lisp is homoiconic, the code you read directly represents the structure of the language [^1]. At it's simplest level, LISP [^2] consists of lists and atoms (which can be numbers, strings, symbols, 'true and NIL.) When you write LISP programs, in a certain way all you are writing are lists which are evaluated according to some simple rules.

For example, when you say `(functionname 1 2 3)` the interpreter/compiler looks up the code corresponding to the first element in the list and applies it to the following elements as arguments.

I don't want to get into things like Macros and Common Lisp Objects because in all honesty I can't speak technically about them with confidence, but in a way conceptually understanding Common Lisp collapses into this understanding. As you read about Common Lisp, the rest will become apparent in time.

Ultimately, there is very little in the toolkit of other programming languages that cannot be implemented explicitly in Common Lisp. The one thing I wonder about off the top of my head is Static Typing, but there have been attempts. While other languages may have richer syntaxes to make useful programming concepts first-order concepts, Lisp allows you to construct them yourself.

I have an intuition that being fluent in Common Lisp will allow me to thing about programming not as syntax and operations, but as concepts and intentions. It may not be the language I ever work in, or perhaps not even the language I always play in, but I feel it will heavily guide the framework on which I think in.

### Alternatives to Common Lisp

What about languages like Racket and Clojure? I think they're pretty cool too, and while I don't want to talk about Functional Programming I do think their FP emphasis will lead people to write better code! They do have different values and priorities which I don't like as much as Common Lisp, but calling them drawbacks at this point would be somewhat unfair. I'm sure I'll have more opinions as time goes by :) 

### Reading Material

The go-to standard for learning Common Lisp is probably Peter Siebel's [Practical Common Lisp][pcl], which is fortunately free. While it is available in dead-tree format, I found its online presentation a little difficult to follow because of the sheer amount of text. It is definitely worth reading, and is probably the most comprehensive introduction to learning Common Lisp.

The book that best helped me grok Common Lisp is the absolutely wonderful and quirky [Land of Lisp][lol], which is worth every penny. Conrad Barski put a lot of work into writing a work which is entertaining and easy to learn from. He uses game programming to teach you both Lisp programming and enough functional programming concepts to orient you around Lisp's thought process. Even if you don't intend on learning Common Lisp itself, [Land of Lisp][lol] is simply wonderful literature alongside a great programming language introduction.

While not a starting point, the definitive online reference of Common Lisp's standard is hosted by LispWorks as the [HyperSpec][lwh]. Paul Graham wrote a more readable reference called [ANSI Common LISP][acl] if dry technical references aren't your style.

There are plenty of other books I'd recommend in the future (Doug Hoyte's [Let over Lambda][lolh], Paul Graham's [On Lisp][pgol]), but for now stick with the above ones.
If you're interested in how Lisps are implemented -- implementation details aren't essential but you can sometimes glean insights into programming language concepts -- I would recommend [Build Your Own Lisp] [^3] and eventually Christian Quinnec's [Lisp in Small Pieces][lisp]

### Recommended Implementations

Obviously, you'll need a Common Lisp interpreter. While a lot of recent languages tend to have a de-facto interpreter or compiler, you will be forced to choose between different Common Lisp variants and there won't be an obvious choice. While this hopefully won't affect your first attempts at writing programs, there's a lot of stuff that wasn't standardized and each implementation has their own set of non-standard extensions and functions. For historical reasons, the chances are low for a second Common Lisp standard being introduced, but at least it won't be as bad as dealing with multiple C or C++ implementations :)

When looking around for an open source Common Lisp implementation, the usual recommendations are either [Steel Bank Common Lisp][sbcl] or [Clozure Common Lisp][ccl]. Conventional wisdom seems to be that the former is faster and smaller, while the latter has some nice OS X bindings and a sliiightly more convenient command line interface[^4]. So far it hasn't mattered for me, although it may turn out that I need some implementation-specific functionality from one that isn't in the other.

There are two extant commercial implementations of Common Lisp: [LispWorks][lw] and [Allegro Common Lisp][facl]. They usually have their own graphical interface, and apparently have proprietary LISP libraries for things like graphical interfaces and databases that cannot be touched by Open Source implementations, but you'll have to decide if their expense and non-free components are worth it to you.

### Editors and Tools

If you go the open source route [^5], the path of least resistance is to use a combination of [Emacs][emacs] (the editor) and [SLIME][slime] (the Superior Lisp Interaction Mode for Emacs). I don't think I'll get into what a REPL is just now, but Lisp's interpreted nature allows for a really well-tuned mechanism for easily evaluating Lisp code in Emacs, code completion, and debugging.

If you aren't into Emacs, I don't blame you. While there are tools for Vim, Eclipse and Sublime Text], I don't know anything about them and I wouldn't recommend anything that's simply a way to use the command-line REPLs in your editor window.

I've tried the [LispWorks][lw] and [Allegro][facl] editors and they're not bad! I prefer Emacs, though.

Regardless of which of these you choose, you will probably want to start out by reading my last [post]({% post_url 2014-10-14-common-lisp-with-tdd-skeleton-organization %}) and installing the [QuickLisp][ql] dependency management at the very least. QuickLisp is the easiest way to install SLIME, if you choose to go that route.

### Have fun!

Hopefully this is enough for you to get started! Hopefully I haven't mislead you!

[^1]: That's at least what they say, I'm sure anyone prefixing a rebuttal with "technically..." will have valid points.
[^2]: LISt Processing, they tell me.
[^3]: This online book is will actually involve programming in C, but that's often what other languages' interpreters and compilers are written in.
[^4]: Not that you want to be using the command line interface by itelf anyway.
[^5]: The Open Source tools totally support the commercial implementations!

[pcl]: http://www.gigamonkeys.com/book/
[lolb]: http://landoflisp.com/
[lwh]: http://www.lispworks.com/documentation/HyperSpec/Front/
[acl]: http://www.paulgraham.com/acl.html
[lolh]: http://letoverlambda.com/
[pgol]: http://www.paulgraham.com/onlisp.html
[byol]: http://www.buildyourownlisp.com/
[lisp]: http://pagesperso-systeme.lip6.fr/Christian.Queinnec/WWW/LiSP.html
[ccl]: http://ccl.clozure.com/
[sbcl]: http://www.sbcl.org/
[facl]: http://franz.com/enterprise_development_tools.lhtml
[lw]: http://www.lispworks.com/news/news31.html
[slime]: http://common-lisp.net/project/slime/
[ql]: http://www.quicklisp.org/
[emacs]: http://www.gnu.org/software/emacs/
