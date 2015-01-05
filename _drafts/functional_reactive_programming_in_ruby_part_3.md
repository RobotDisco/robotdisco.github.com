---
title: Functional Reactive Programming in Ruby (Part 3)
layout: post
date: 2015-01-04
---

### Last time(s)
I talked about functional reactive programming and how I felt it would be useful in building a roguelike video game in Ruby.

If you’re interested in the code, it’s over [here](http://github.com/NaleagDeco/ykwya-roguelike). My apologies if it isn’t as easily deployable as it should be.

Anyway, let’s get to it <3

### Creating the world Functionally
Before we even think about input and output, let’s think the functional programming. I wanted to have monsters, potions, gold, a player, and some terrain. I also want a status message to be displayed to the player. You’ll want to look at the code for the full story, but here’s the crux of my code for creating a new game.

{% highlight ruby linenos %}
def self.create_game(player)

  terrain = build_dungeon
  unplaced_monsters = build_monsters
  unplaced_potions = build_potions
  unplaced_gold = build_gold
  
	# Shuffle to enable random placement
  empty_terrain = terrain.select { |coords, tile| tile.is_a? YKWYA::Empty }.keys.shuffle

  {
    player: player.clone,
    terrain: terrain,
    player_coords: empty_terrain.pop,
    stairway_coords: empty_terrain.pop,
    potions: empty_terrain.pop(unplaced_potions.size).zip(unplaced_potions),
    monsters: empty_terrain.pop(unplaced_monsters.size).zip(unplaced_monsters),
    gold: empty_terrain.pop(unplaced_gold.size).zip(unplaced_gold),
    status: 'Welcome to YKWYA!'
  }
end
{% endhighlight %}[^1]

[^1]: This isn’t my usual ruby style, but it seemed easiest to read.

Doesn’t it look really weird? (It probably looks even weirder if you’re comfortable in Ruby. Because I’m trying to minimize the amount of state changes I make, I’m not writing idiomatic Ruby. In many ways, my code reminds me of Scala, which attempts to put a functional-friendly syntax on top of Java but is basically just Java.

In essence, I’m
1. Creating the terrain, monsters, etc… Usually this was a list of generated objects and the coordinates they’d live on. In the case of the terrain, it was a giant hash table because I didn’t feel like doing anything fancier.
2. Put everything into a giant hash table, and return it!

Pretty simple, eh? (The fun stuff is in how you generate everything :P)

You’ll notice that I passed the player in as an argument and cloned it. I could have easily just created the player object inside, but I wanted my character creation logic to create the player first so users had choice of what kind of character they played.
It’s actually kind of handy, because that `.clone` is important:  Since Ruby doesn’t copy objects, I don’t want to pass that original player b/c then I might accidentally modify it from multiple parts of my code. You’ll see `.clone` all over my code, which is my fault for using a language not designed for this.

### Modifying the world functionally
Ultimately, every turn of my simple roguelike consists of two steps:
1. The player performs an act (gets potion or gold, attacks a player, moves, or doesn’t move)
2. All monsters act (move around or attack.)

Like all complex software, this won’t necessarily be super easy. You’ll wind up having to juggle a bunch of different pieces of data that depend on each other, but the nice thing is that all of this data is collected into one object that you can pass around from function to function.

I’ve found that it’s a lot easier if you break all of your operations into little chunks:
* Does the player move or attack
* What happens when the player picks up an item / attacks a monster
* Did the world change (a door opened, a trap activated)
* What do the monsters do?

I have no strong design basis for this, but I’d also recommend you use functions that generally take in the entire game state object and return one as well. This allows you to always have all the dependencies you need and you can then write all logic as a chain of calls that are always consistent at any step.

### Next time
We’ll implement the reactive bit so you can type commands and see things!