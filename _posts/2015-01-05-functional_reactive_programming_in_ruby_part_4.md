---
title: Functional Reactive Programming in Ruby (Part 4)
layout: post
date: 2015-01-05 10:50

---

### Last time(s)
We talked about implementing a basic roguelike in a functional-programming kind of way.

### An admission.
This is me figuring stuff out, so it may turn out I’m not doing things in the best way! It’s worked for me so far, though.
This is actually the part that made me want to write a series of posts, because I spent so much time trying to tie all of this stuff together because it wasn’t (at the time) clear to me how to do all of this together.

### How we’d do it in a all-the-way FRP environment
Now we’ll wrap it up to make it reactive. If I had been writing this in a self-contained FRP environment like [Elm](http://www.elm-lang.org) I would have probably had a bunch of signals available to me and environment would probably naturally expect a signal of something “drawable”[^1] that would then automagically be rendered to the screen

[^1] What that “drawable” thing is depends on the system you’re using, I guess what I’m saying is that for you it would be the endpoint.

### Frappuccino
Sadly, or happily, for a bunch of reasons I wrote this in Ruby so I don’t have the advantage of having that all done for me. I have to find a reactive library I can leverage.

As I was looking around for reactive libraries to use in Ruby (Ruby does an awesome job by itself for functional programming, don’t really need any additions there) I wound up settling on [Frappuccino](http://github.com/steveklabnik/frappuccino) because it seemed the most comfortable to me.

### Curses!
So the first thing we’re going to need is an input event signal. 

Well actually, first I’m going to have to get input from the screen in the first place.

Traditionally Roguelikes have used an ASCII library called “Curses” to make their distinctive “everything-is-a-keyboard-character” look. I did not feel like deviating from that, so I’m using [Ruby’s implementation](http://www.ruby-doc.org/stdlib-2.0/libdoc/curses/rdoc/Curses.html)

The very superficial explanation is that you create a “screen” object, which has methods for drawing characters to the screen and getting keypresses. I’m going to wrap these in my code, hopefully they aren’t too confusing to skip around.

### At a high level
Here’s what the core loop of my code looks like

{% highlight ruby %}
while true
  execute_command! @main.getch
end
{% endhighlight %}

It’s not very functional :( But this code is all about input and output, so I don’t feel too bad :)

You’ll see that the core of my game is all about waiting for new input. the `getch` method reads a keypress from the screen, which gets passed into the `execute_command!` method.

(How does the screen update, you ask … keep reading!)

### Input

{% highlight ruby %}
    def execute_command!(char)
      case char
      when 'h'
        YKWYA::user_input.emit :move_left
      when 'j'
        YKWYA::user_input.emit :move_right
…
      when 'w'
        YKWYA::user_input.emit :wait
      when 'q'
        YKWYA::user_input.emit :quit
      end
    end
{% endhighlight %}

This code is a bit ugly. Basically, for every key that I want to turn into an action, I have an object called `user_input` that ‘emits’ a symbol[^2]. What is `user_input`? It’s an object that feeds into a signal![^3]

[^2]: Symbols are somewhat hard to explain if you’ve never used Lisp or Ruby, but they’re super awesome! They’re little throwaway things you can match against. You might use strings or integers in other languages but they don’t take up as much memory as strings and you can’t accidentally use them for strings (unlike enums in C where you can all-too-easily treat them like a number.) In another language I would have just passed a string, but the code is more readable and has some performance benefits.

[^3]: This is a little clunky because of the library I’m using and some Ruby weirdness. Signals in this library must get input from because of Ruby’s object-oriented nature. When I tell a signal to listen to a particular object, it automatically gives that object an emit method. This was really confusing when I started, I didn’t know where the emit function came from!

If you wanted to do a game that wasn’t turn-based, you’d probably need to do something more complicated where the a no-input symbol was emitted by default. Luckily I did not have that problem, it was okay for the game to be paused while it waited for a keypress!

### Advancing the game
Remember that fold or reduce operation I [told you about]({% post_url 2015-01-04-functional_reactive_programming_in_ruby_part_2 %})? Well, I use it to my advantage here!

{% highlight ruby %}
  @input_stream = Frappuccino::Stream.new(user_input)
  initial_state = YKWYA::GameState.create_game(player)
  game_state = @input_stream.scan(initial_state) do |last_state, input|
     YKWYA::GameState.process_input(last_state, input)
  end
{% endhighlight %}

A couple clarifying points:

* `user_input` is an plain object with nothing special in it. You saw me call emit on it in an earlier code block, but maybe I’d want to write special methods for code readability or portability or something.
* The Frappuccino library calls signals streams. Just read it as signal! There are some technical differences that aren’t important for my purposes (mostly that time isn’t inherently involved unless I manually made the stream update periodically. For our purposes this is a perfectly fine event signal.)

This is part of my program startup code. Before I enter the above loop, I create a signal and tell it to listen to `user_input`. I create a player in character creation and pass it to to the function which creates my initial game state. Then I use the `.scan` function to create a _new_ stream which starts off with the initial game state and then, based on the value of the input stream as new inputs come in, produces a new game state (which reflects whatever actions the player was told to do.)

The upshot of this is that the `game_state` stream now always has the latest game_state, and if we wanted to we could pieces together the complete history of the game from the moment it was run :D

### Output
But how do we render to the screen?

Frappuccino has a special callback feature we can use:

{% highlight ruby %}
  game_state.on_value do |state|
    render! state
  end
{% endhighlight %}

Every time the game_state signal updates, we tell Curses to draw the screen based on the new game_state. It’s that simple!

### Conclusion
And with that, we have a roguelike where the input very simply gets fed into a signal, the output automatically gets triggered by the signal, and you can put all the work into where it should be, the logic of the roguelike you are trying to build. This pattern, if you can find an FRP environment for your requirements, is probably the easiest way I’ve seen of handling user interface loops!
