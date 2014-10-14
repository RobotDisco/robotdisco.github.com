---
layout: post
title: "A project structure skeleton for test-driven Common Lisp"
categories: common-lisp
---

*Editor's note: Given how I new I am to this language and ecosystem, I wouldn't be surprised to find out that I made some bad choices as I stumble around orienting myself. I'll try my best to revisit this page as I get a better understanding of what I'm doing.*

I'm relatively new to Common Lisp programming; I can write simple programs and have the basic language down, but I haven't done anything significantly complex in it.

I love the language, but I've found that part of Common Lisp's 80s heritage is that there are toy examples or real (i.e. messy) projects and not much in between. Common Lisp is also a return to the "there's more than one day to do it!" paradigm, unlike many of the newer languages I've been writing recently. It's strange how the same things that aggravate me about C (a flood of implementations, platform-dependent ambiguities, low-levelisms) are romantic in Common Lisp.

Playing around in Emacs and SLIME was pretty easy, but trying to organize my code into a code and tests was disorienting and there wasn't particularly accessible documentation around the concepts of systems and packages. The documentation that exists reads with the same dryness of the GNU manual, although I'll probably appreciate that as I understand more.

For now, I'm going to try to lay out a simple Common Lisp project layout for test driven development in the hopes that other people won't have to go searching around the way I did.



#### The Basics
---

First off, I've tried this in both the [SBCL][sbcl] and [Clozure Common Lisp][ccl] implementations, which seem to be the most recommended. I highly recommend you download and install Zachary Beane's [QuickLisp][ql] for dependency management.

Zachary also makes [QuickProject][qp], a nice tool for producing simple Common Lisp project skeletons. It and the [cl-git][cl-git] project were what I wound up basing my project structures on.

Assuming you have QuickLisp installed, you can create a skeleton project in a REPL as follows:

{% highlight common-lisp %}
(ql:quickload :quickproject)
(quickproject:make-project #p"~/Code/my-project")
{% endhighlight %}

The quickload command will import the referenced project, downloading it if necessary. The `#p` string prefix is supposed to indicate path names in a supposedly portable way, but regular strings worked as well for me. Maybe the behaviour is more significant for Windows?

#### Directory Structure
---

Quickproject generates the following files:

* README.txt
* my-project.lisp (This would be where your code would go)
* package.lisp (A file that defines the project's Common Lisp `package`. More on this later.)
* my-project.asd (A file that define's the project's `system`. More on this later.)

This would be all well and good if I didn't want to separate things into testing and logic components. The following is the format I decided to go with, based on browsing over a bunch of Common Lisp projects and guessing at best practices:

* README.txt
* package.asd
* t/
  * test1.lisp
  * test2.lisp
* src/
  * package.lisp
  * one-module.lisp
  * another-module.lisp

`t` and `src` seem to be conventional directory names for test and source code, so I've stuck with them. In most non-trivial codebases I would want to separate my code into modules, so I've shown that as well.

#### Packages and Systems
---

Packages and Systems seem to be important concepts in the Common Lisp ecosystem. They are related, but they aren't quite the same. I have seen people define their packages in `package.lisp` files, in the project's ASD files, and as preamble in the main project logic files. I've chosen to stick with `package.lisp` until I have a good reason not to.

A `package` is, as far as I can tell, a namespace. We would want different namespaces for our different modules, even if they are all bundled together in a project for distribution. So far I haven't needed a package definition in my test files because I'm always using the ones defined in `src/`.

Given that I have the modules listed above, my src/package.lisp file would look like:

{% highlight common-lisp %}
(defpackage #:myproject-one
  (:use #:cl))

(defpackage #:myproject-another
  (:use #:cl))
{% endhighlight %}

Dashes seem to be the way you qualify subpackages. Unlike some languages, package names are not tied to your directory structure or any other metadata. The `(:use #:cl)` merges in the standard Common Lisp namespace and appears to be convention. I haven't seen what damage occurs if I omit it :P


A `system` is how you configure the package for distribution; Common Lisp programs itself don't care about them (they care about namespaces) but the system concept was created in a third party library that seems to be the standard distribution system that tools like QuickLisp conform to. The ASD file is just a Lisp file that contains a manifest of the project's contents. Eventually you will want to read up on the ASDF system, but the following is what I cobbled together for my desired project structure.

{% highlight common-lisp %}
(asdf:defsystem #:myproject
  :description "My example project"
  :author "Gaelan D'costa"
  :license "MIT"
  :serial t
  :pathname "src/"
  :components ((:file "package")
               (:file "on-module")
               (:file "another-module")))

(asdf:defsystem #:myproject-tests
  :description "Tests for my example project"
  :author "Gaelan D'costa"
  :license "MIT"
  :serial t
  :depends-on (:FiveAM :myproject)
  :pathname "t/"
  :components ((:file "test1")
               (:file "test2")))
{% endhighlight %}

Most of those fields should be fairly straightforward. The only tricky keywords are `:components`, which can be more than just a list of files, and `:serial`, which indicates that each entry in `:components` should depend on all previous entries. `:FiveAM` is the package of the unit test framework I am using, and obviously I'll want to import the system I'm testing.

Why am I creating two systems? I don't have a great answer for that other than an intuitive feeling that other people don't want to import test libraries if all they want to do is use my stuff. I could take advantage of the `:component` keyword and have everything in one system if I wanted to, it's definitely something that people do.

#### Finishing up
---

With this, I now have a project that I can import into my REPL to either work on or use as a dependency for other projects. Eventually I may even want to upload my work to QuickLisp's repository if I want to share it with the world.

The QuickLisp site has instructions for including local projects in its index, but when starting a new project I sometimes don't want to permanently import it just yet. If I use the expression `(load #p"~/path/to/my/project/myproject.asd")` then I will be able to `(ql:quickload :myproject)` for the current REPL section.

[ql]: http://http://www.quicklisp.org/
[qp]: https://github.com/xach/quickproject
[cl-git]: https://github.com/russell/cl-git/
[ccl]: http://ccl.clozure.com/
[sbcp]: http://www.sbcl.org/
