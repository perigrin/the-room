---
layout: post
title: "Part 2 - Mazes and Monsters"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image: https://live.staticflickr.com/65535/52126535318_da553d66cb_5k.jpg
---

## Introduction

This post is part of a series of blog posts following the [roguelike
tutorial](https://www.rogueliketutorials.com/) to demonstrate the new `class`
feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part-1](). If you don't
remember what the code looked like, go back and refresh yourself. From now on
the listing is gonna get a little large for us to copy it every time.

We're going to making our game world a little more interesting than a player
wandering a blank void. We'll be adding other characters and giving our player
somewhere to walk around in.

### Making new Friends

We left off able to render the character's icon and move it around a blank
screen, which is only impressive or exciting for about two minutes. Let's add
some non-player characters (NPCs), or as we’ll call them: Entities.

Let's start by creating a new class right before our Engine class.

```
class Entity { }

class Engine {
	[...]
}
```

We know that our Entities will need a few things, they'll need a location, some
kind of icon, and let's give them unique colors too.

```

class Entity {
	field $x :param;
	field $y :param;
	field $char :param;
	field $fg :param //= '#fff';
	field $bg :param //= '#000';
}

```

If you notice for our foreground (fg) and background (bg) colors we used Perl's
“defined or” assignment to set some defaults. This means that if those
parameters aren't passed into the constructor we'll use these values instead.

We're also gonna need a way to look at these values, so we'll need to create
some methods for that.

```
class Entity {
	field $x :param;
	field $y :param;
	field $char :param;
	field $fg :param //= '#fff';
	field $bg :param //= '#000';

	method x { $x }
	method y { $y }
	method char { $char }
	method fg { $fg }
	method bg { $bg }
}

```

Remember back in part-0 where I talked about how the new class system in Perl
was designed to default to encapsulation? This is an example of that.If we
don't explicitly make a way to access the variables, they really shouldn't be
easily visible outside the scope of the class. In Perl's traditional object
system, every object was an explicit data structure, usually a hash ref, and
people would routinely access the data the way they would any other kind of
data structure.

About now you're probably asking yourself “Why do I care about this?” If you're
not, then you can skip ahead a little. Otherwise, let's take a second and
discuss why we want to encapsulate the implementation in objects.

One of the main purposes for object oriented programming was to allow for a
natural way to promote isolation and loose coupling in code. In simpler terms,
it's meant to allow you to limit the number of places you need to change code
when you need to add new behavior or fix bugs. The methods of an object become
a contract with the code outside of the class that says "if you call render(),
you don't need to worry about how render() is accomplished, just know that it
will be.” We'll see this play out as we move along refactoring and adapting our
classes as the game gets more complex.

With the traditional way of treating objects as just another data structure,
sometimes people could put the object into a state it wasn't expecting and
cause bugs. It encouraged tighter coupling of code inside and outside of the
object, and made it so that you needed to be more careful with the changes you
made.

Let's get back to our `Entity`, one more thing we want entities to be able to
do is move about, so let's add a move method that updates their location:

```
class Entity {
	field $x :param;
	field $y :param;
	field $char: param;
	field $fg :param //= '#fff';
	field $bg :param //= '#000';

	method x { $x }
	method y { $y }
	method char { $char }
	method fg { $fg }
	method bg { $bg }

	method move($dx, $dy) {
    	$x += $dx;
    	$y += $dy;
	}
}
```

This should give us a basic entity object. Let's refactor our code to use it
for our player first.

```
class Engine {
	field $height :param;
	field $width: param;

	field $player = Entity->new(
   	x	=> $width / 2,
   	y	=> $height / 2;
   	char => '@',
   	fg   => '#fff',
   	bg   => '#000',
	);

	field $app = Games::ROT->new(
    	screen_width  => $width,
    	screen_height => $height,
	);

	[...]

}
```

We replace `$player_x` and `$player_y` with a new field `$player` that is an
Entity object. We'll need to update our render method as well.It should now
look something like this:

```
method render() {
	$app->clear();
	$app->draw(
    	$player->x,
    	$player->y,
    	$player->char,
    	$player->fg,
    	$player->bg
	);
}
```

This is a little bit more verbose, but we're gonna get those extra lines back
when we start adding more entities for NPCs. Lastly, we're gonna need to update
our movement handlers:

```
	my %KEY_MAP = (
    	h => sub { $player->move(-1,0) },
    	j => sub { $player->move(0,-1) },
    	k => sub { $player->move(0,1)  },
    	l => sub { $player->move(1,0)  },
    	q => sub { exit }
	);

```

If we run this, everything should look exactly the same. That's the sign of a
good refactor, nothing changed.

We're still by ourselves though, so let's add some more entities. Just after
the `$player` field declaration let's add a new one for `@npcs`

```
class Engine {
	field $height :param;
	field $width: param;

	field $player = Entity->new(
   	x	=> $width / 2,
   	y	=> $height / 2;
   	char => '@',
   	fg   => '#fff',
   	bg   => '#000',
	);

	field @npcs = (
    	Entity->new(
        	x => $width / 2 - 5,
        	y => $height / 2,
        	char => 'N',
        	fg => '#0f0',
        	bg => '#000',
    	),
	);

	[...]

}
```

If we run things with this change, we're still just gonna see our player all
alone in a void. Let's adjust the `render` method.

```
method render() {
	$app->clear();
	for my $e (@npcs, $player) {
    	$app->draw($e->x, $e->y, $e->char, $e->fg, $e->bg);
	}
}
```

We're getting there. If we run things now we'll have our player and a new
friend, Mr. N, nearby. But they're still standing in a void. We need to build a
world around them, traditionally this is called the Game Map, so let's create a
class for that.

Just above the Entity class let's add a new `GameMap` class

```
class GameMap { }

class Entity {
	field $x :param;
	field $y :param;
	field $char: param;
	field $fg :param //= '#fff';
	field $bg :param //= '#000';

	method x { $x }
	method y { $y }
	method char { $char }
	method fg { $fg }
	method bg { $bg }

	method move($dx, $dy) {
    	$x += $dx;
    	$y += $dy;
	}
}

```

We know that the `GameMap` has a height and a width. So we can add fields for
those, and a method to check if we're in the bounds.

```
class GameMap {
	field $height :param;
	field $width :param;

	method is_in_bounds($x, $y) {
    	0 <= $x < $width && 0 <= $y < $height
	}
}
```

A `GameMap` is also traditionally made up of Tiles, which know whether the tile
is “walkable” (True if it’s a floor, False if its a wall),  “transparent”
(again, True for floors, False for walls), and how to render the tile to the
screen. We don't currently have Tiles, so let's leave the `GameMap` for a second
and build ourselves a Tile class.

```
class Tile {
	field $walkable	:param;
	field $transparent :param;
	field $char    	:param //= '';
	field $fg      	:param //= '#fff';
	field $bg      	:param //= '#000';

	method is_walkable() { $walkable }
	method is_transparent { $transparent }
	method char() { $char }
	method fg() { $fg }
	method bg() { $bg }
}

```

This is pretty straight forward, we have fields for all of the values we care
about and reader methods for them. Now that we can have tiles, let's set up our
`GameMap`.

```
GameMap {
	field $height :param;
	field $width :param;

	field @tiles;

	my sub FLOOR_TILE() {
    	Tile->new(
        	walkable	=> 1,
        	transparent => 1,
        	char    	=> '.',
        	fg      	=> '#333',
    	);
	}

	ADJUST {
    	# draw the floor
    	@tiles = map { [map { FLOOR_TILE() } 0..$width] } 0..$height;
	}
}
```

We create a new function for creating floor tiles, using `my` to make sure that
it's lexically scoped to inside the `GameMap`, which is the only thing that
really cares about Tiles, we then use that to populate the tiles with a
multi-dimensional array of floor tiles.

We’ll also add a `render` method to the map to allow it to draw itself.

```
field @tiles = map { [map { FLOOR_TILE() } 0..$width] } 0..$height;

method render($term) {
	for my $y (0..$#tiles) {
    	my @row = $tiles[$y]->@*;
    	for my $x (0..$#row) {
        	my $tile = $row[$x];
        	$term->draw($x, $y, $tile->char, $tile->fg, $tile->bg)
    	}
	}
}
```

Now let's update the Engine to know about the `GameMap`. First we add a field
for it.

```
class Engine {
	field $height :param;
	field $width: param;

	field $map = GameMap->new(
    	height => $height,
    	width  => $width,
	);

	field $player = Entity->new(
	[...]
)
```

Then we update Engine's `render` method to call the one in the map.

```
method render() {
	$app->clear();
	$map->render($app);
	for my $e (@npcs, $player) {
    	$app->draw($e->x, $e->y, $e->char, $e->fg, $e->bg);
	}
}
```

Now in theory if we've gotten everything right and we run the app, we should
have our two characters on a field of floor tiles. Slightly better than a void,
but let's add a wall for them to play with.

Inside our `GameMap` class let's add a function to generate wall tiles, and
update the ADJUST block to add some.

```
my sub WALL_TILE() {
	Tile->new(
    	walkable	=> 0,
    	transparent => 0,
    	char    	=> '#',
    	bg      	=> '#000064',
	);
}

field @tiles = map { [map { FLOOR_TILE() } 0..$width] } 0..$height;

ADJUST {
	# draw a little wall in the room
	@tiles[22][30..32] = map { WALL_TILE() } (1..3);
}
```

Now if you start things up you should see a little wall drawn on the game
board. Unfortunately, we can apparently walk through walls. We haven't invented
a way to die yet, so I'm _pretty_ sure we're not a ghost, we should fix that.

Let's update our movement actions.

```
class Engine {
[...]

	ADJUST {
    	my sub movement_action($dx, $dy) {
        	my ($x, $y) = ($player->x + $dx, $player->y + $dy);
        	return if !$map->is_in_bounds($x, $y);
        	return if !$map->tile_at($dx, $dy)->is_walkable;
        	$player->move($dx,$dy);
    	}

    	$app->add_event_handler(
        	'keydown' => sub ($event) {
            	my %KEY_MAP = (
                	h => sub { movement_action(-1, 0) },
                	j => sub { movement_action( 0,-1) },
                	k => sub { movement_action( 0, 1) },
                	l => sub { movement_action(-1, 0) },
                	q => sub { exit }
            	);
            	# lets execute the action now
            	$KEY_MAP{$event->key}->();
        	}
    	);
    	$app->run( sub { $self->render() } );
	}

	my sub handle_input($event) {
    	my %KEY_MAP = (
    	);

    	return $MOVE_KEYS{$event->key};
}
```

The movement action is getting a little complicated, so we turned it into a
private subroutine for now, but when things get complicated later I don't think
this will continue to work. Also,we've added some sanity checks before we have
the player actually move. We'll need to add the `tile_at` method to `GameMap`
so that those checks will actually work.

```
class GameMap {
	field $width   :param;
	field $height  :param;

	field @tiles = ([]);

	method tile_at($x, $y) { $tiles[$y][$x] }

	[...]
}
```

Now we should be prevented from walking through walls. With that we've finished
making the bare bones of NPCs and walls, we have all the basics in place to
generate dungeons to explore, we'll do that next time!

Here's the complete listing of our code to date.

```
#!/usr/bin/env perl
use 5.38.0;
use warnings;

use lib qw(lib);
use experimental 'class';

use Games::ROT;

class Tile {
	field $walkable :param;
	field $transparent :param;
	field $char :param //= '';
	field $fg :param //= '#fff';
	field $bg :param //= '#000';

	method is_walkable() { $walkable }
	method is_transparent { $transparent }
	method char() { $char }
	method fg() { $fg }
	method bg() { $bg }
}

class GameMap {
	field $width   :param;
	field $height  :param;

	method is_in_bounds($x, $y) {
    	return 0 <= $x < $width && 0 <= $y < $height;
	}

	field @tiles;

	my sub FLOOR_TILE() {
    	Tile->new(
        	walkable	=> 1,
        	transparent => 1,
        	char    	=> '.',
        	fg      	=> '#333'
    	);
	}

	my sub WALL_TILE() {
    	Tile->new(
        	walkable	=> 0,
        	transparent => 0,
        	char    	=> '#',
    	);
	}

	ADJUST {
    	# draw the floor
    	@tiles = map { [map { FLOOR_TILE() } 0..$width] } 0..$height;

    	# draw a little wall in the room
    	$tiles[22]->@[30..32] = map { WALL_TILE() } 0..2;
	}

	method render($term) {
    	state $i = 0;
    	for my $y (0..$height) {
        	for my $x (0..$width) {
            	my $tile = $self->tile_at($x, $y);
            	$term->draw($x, $y, $tile->char, $tile->fg, $tile->bg);
        	}
    	}
	}

	method tile_at($x, $y) {
    	return $tiles[$y][$x];
	}
}

class Entity {
	field $x :param;
	field $y :param;
	field $char: param;
	field $fg :param //= '#fff';
	field $bg :param //= '#000';

	method x { $x }
	method y { $y }
	method char { $char }
	method fg { $fg }
	method bg { $bg }

	method move($dx, $dy) {
    	$x += $dx;
    	$y += $dy;
	}
}

class Engine {
	field $height :param;
	field $width :param;

	field $player = Entity->new(
   	x	=> $width / 2,
   	y	=> $height / 2,
   	char => '@',
   	fg   => '#fff',
   	bg   => '#000',
	);

	field @npcs = (
    	Entity->new(
        	x => $player->x - 5,
        	y => $player->y,
        	char => 'N',
        	fg => '#0f0',
        	bg => '#000',
    	),
	);

	field $app = Games::ROT->new(
    	screen_width  => $width,
    	screen_height => $height,
	);

	field $map = GameMap->new(
    	width   => $width,
    	height  => $height,
	);

	ADJUST {
    	my sub movement_action($dx, $dy) {
        	my ($x, $y) = ($player->x + $dx, $player->y + $dy);
        	return unless $map->is_in_bounds($x, $y);
        	return unless $map->tile_at($x, $y)->is_walkable;
        	$player->move($dx,$dy);
    	}

    	$app->add_event_handler(
        	'keydown' => sub ($event) {
            	my %KEY_MAP = (
                	h => sub { movement_action(-1, 0) },
                	j => sub { movement_action( 0, 1) },
                	k => sub { movement_action( 0,-1) },
                	l => sub { movement_action( 1, 0) },
                	q => sub { exit }
            	);
            	# lets execute the action now
            	$KEY_MAP{$event->key}->();
        	}
    	);
    	$app->run( sub { $self->render() } );
	}

	method render() {
    	$app->clear();
    	$map->render($app);
    	for my $e (@npcs, $player) {
        	$app->draw($e->x, $e->y, $e->char, $e->fg, $e->bg);
    	}
	}
}

my $engine = Engine->new(width => 80, height => 50);
```
