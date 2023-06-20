---
layout: post
title: "Part 3 - Mastering the Generative Dungeon"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image:
---


## Introduction

This post is part of a series of blog posts following the [roguelike
tutorial](https://www.rogueliketutorials.com/) to demonstrate the new `class`
feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part-1](). If you don't
remember what the code looked like go back and refresh yourself.

When last we left our intrepid characters they were in a nearly empty room with
a tiny wall in the middle. This is something but it's not really an adventure
game. We want to have dungeons to explore! In this post we'll build ourself a
dungeon generator.

### Before We Begin

Before we get started things are starting to get a little ungainly. We've been
keeping everything in one file, and that makes for quick development but it's
getting harder to see what's going on. Let's break things apart a little.

Let's create ourselves a `lib` directory and make a `GameMap.pm` inside of it.
Just like in our main file we'll need to tell Perl to use 5.38, and to enable
the warnings pragma and the class feature.

```
use 5.38.0;
use warnings;
use experimental 'class';
```

Now we move the `Tile` and `GameMap` classes to it in their entirety. Repeat
this process with the `Entity` class into an `Entities.pm` and the `Engine`
class into an `Engine.pm`. In `Engine.pm` you'll also want to move  the `use
Games::ROT;` line to load that library.

When you're done you should a directory now that looks something like this:

```
.
├── game.pl
└── lib
    ├── Engine.pm
    ├── Entities.pm
    └── GameMap.pm
```

### Getting Started

Let's get rid of that wall, we're not going to need it if we're building a
whole dungeon. We can just remove the ADJUST block in the GameMap class.

```
    [...]
    field @tiles = map { [map { WALL_TILE() } 0..$width] } 0..$height;

    # ADJUST BLOCK WAS HERE

    method render($term) {
    [...]
```

Now we could stuff all the dungeon generation code into the GameMap class but
depending on how you look at it a dungeon can be made up of many maps (one per
level!). If we add multiple kinds of dungeon generators then the map class will
get potentially huge with the business of generating dungeons not with the
business of _being_ a map. Instead, in the words of those classical poets The
Offspring, we're going to keep 'em separated.

Let's create a `ProcGen.pm` file in our lib directory. We're going to start by
creating a class to represent rooms. In our new file let's create a
RectangularRoom class.

```
use 5.38.0;
use warnings;
use experimental 'class';

clss RectangularRoom {
    field $x1 :param('x');
    field $y1 :param('y');
    field $height :param;
    field $width :param;

    field $x2 = $x1 + $width;
    field $y2 = $y1 + $height;

    method center() {
        my $x = int($x1 + $width / 2);
        my $y = int($y1 + $height / 2);
        return [$x, $y];
    }

    method inner() {
        [$x1 + 1 .. $x2], [$y1 + 1 .. $y2];
    }
}
```

Obviously rooms need a starting point and a height and width, to make things
easier on ourselves we also calculate the opposite point from the origin. We
want to call our origin fields `$x1` and `$y1` but those names don't really
make sense outside of our class structure. The `:param` attribute can take an
argument to specify what the constructor argument should be named, so we tell
it to use `x` and `y` respectively. This is another example of encapsulating
the implementation details from the outside world.

The `center` method returns a tuple (a fancy word for an array ref of two
elements) for the coordinates of the center of the room. The `inner` method
returns two arrays of the points for the width and height of the inside of the
room.

We offset the room by 1 (`$x1 + 1`, `$y1 + 1`) to ensure we always have at
least one wall between rooms unless we specifically choose overlap them.

Before we go too crazy with a fully procedurally generated dungeon, let's start
with something simple like two rooms. Let's add a SimpleDungeonGenerator class
just after the RectangularRoom class, with a `generate_dungeon` method.

```
[...]
}

class SimpleDungeonGenerator {
    use GameMap;

    field $width :param;
    field $height :param;

    sub generate_dungeon() {
        my $map = GameMap->new(
            width  => $width,
            height => $height,
        );

        my $room_1 = RectangularRoom->new(
            x => 20,
            y => 15,
            width => 10,
            heigh => 15,
        );
        my $room_2 = RectangularRoom->new(
            x => 35,
            y => 15,
            width => 10,
            heigh => 15,
        );

        $map->_tiles_to_floor($room_1->inner);
        $map->_tiles_to_floor($room_2->inner);
    }
}
```

This is where the defaults toward encapsulation make our lives a little more
difficult. We need a way to modify the tiles inside the GameMap object but we
don't actually expose those tiles. For now we'll create a method with a leading
underscore because Perl programmers have been trained for generations to
pretend those are private.

Let's add that to our GameMap class.

```
method _tiles_to_floor($x_slice, $y_slice) {
    # for every row in the $y_slice
    for my $y (@$y_slice) {
        # convert the $x_slice columns to FLOOR_TILE()s
        $tiles[$y]->@[@$x_slice] = map FLOOR_TILE(), @$x_slice;
    }
}
```

Do I like this solution? No. However it works for now and we can revisit this when we have a better solution. Now let's update our Engine to use this new dungeon generation function.

```
[...]
field $app = Games::ROT->new(
    screen_width  => $width,
    screen_height => $height,
);

field $map = SimpleDungeonGenerator->new(
    width => $width,
    height => $height
)->generate_dungeon();

ADJUST {
[...]
```
If we run the code now we will have two different rooms, but no way to walk
between them. Also poor Mr. N is trapped in a wall. Add the following function to just before `generate_dungeon`.

```
my sub tunnel_between($map, $start, $end) {
	my ($x1, $y1) = @$start;
	my ($x2, $y2) = @$end;
	warn "$x1, $y1 -> $x2, $y2";
	if (rand() < 0.5) {
		$map->_tiles_to_floor([min($x1, $x2)..max($x1, $x2)],[$y1]);
		$map->_tiles_to_floor([$x2],[min($y1, $y2)..max($y1, $y2)]);
	} else {
		$map->_tiles_to_floor([min($x1, $x2)..max($x1, $x2)],[$y2]);
		$map->_tiles_to_floor([$x1],[min($y1, $y2)..max($y1, $y2)]);
	}
}
```

This is a little complicated but what we're doing is we have a 50/50 shot of
either drawing a vertical hallway and then a horizontal hallway from the
starting point, or a horizontal hallway and then vertical one.

We are using `min` and `max` from `List::Util` now, so we'll have to use that package at the top of the class.

```
class SimpleDungeonGenerator {
    use GameMap;
    use List::Util qw(min max any);
```

Now we just update our `generate_dungeon` function to use the `tunnel_between` function.

```
$map->_tiles_to_floor($room_1->inner);
$map->_tiles_to_floor($room_2->inner);

tunnel_between($map, $room_1->center, $room_2->center);
return $map;
```

Presto, a room with a hallway.

No, I agree that's totally not a dungeon is it. But this is nearly all the
pieces we need. We need a way to see if one room intersects with another. Let's
add a method to our RectangularRoom to check for this.

```
method intersects($other) {
	if (ref $other eq 'HASH') {
		if (
			$x2 < $other->{x1} ||
			$y2 < $other->{y1} ||
			$other->{x2} < $x1 ||
			$other->{y2} < $y1
		) { return 0 }
		return 1
	}

    if ($other isa RectangularRoom) {
        return $other->intersects({
            x1 => $x1,
            y1 => $y1,
            x2 => $x2,
            y2 => $y2,
        });
    }

    die "$other is not a RectangularRoom object";
}
```

This seems a little complicated, but this maintains the encapsulation of the
rectangular room object. If we're provided another RectangularRoom object then
we call the intersects method on it and pass it our vertices. If we're given a
HASH ref of vertices we do the comparison and return that instead. This is
designed so that nothing outside the RectangularRoom class need know what shape
it is ... I mean other than the class name, which is a little on the nose. We
could easily replace the HASH ref with a list of vertices and implement
something like a [Weiler–Atherton clipping algorithm](https://en.wikipedia.org/wiki/Weiler%E2%80%93Atherton_clipping_algorithm) and have a more general solution for polygonal rooms, not just rectangles.

Okay let's finally update the generate_dungeon function

```
field $width    :param;
field $height   :param;
field $rooms    :param(room_count);
field $min_size :param(min_room_size);
field $max_size :param(max_room_size);
field $player   :param;

my sub tunnel_between($map, $start, $end) {
    [...]
}

method generate_dungeon() {
    my $map = GameMap->new(width  => $width, height => $height);

    my @rooms;
    for (0..$rooms) {
        my $room_width = int($min_size + rand($max_size + 1 - $min_size));
        my $room_height = int($min_size + rand($max_size + 1 - $min_size));

        my $room = RectangularRoom->new(
            x => int(0 + rand($width - $room_width)),
            y => int(0 + rand($height - $room_height)),
            width  => $room_width,
            height => $room_height,
        );

        # if the new room intersects with a current room, skip it
        next if any { $_->intersects($room) } @rooms;

        # otherwise, dig out the floor
        $map->_tiles_to_floor($room->inner);

        # tunnel between the previous room and this one
        warn "Rooms ".scalar @rooms;
        tunnel_between($map, $rooms[-1]->center, $room->center) if @rooms;

        # add the room to the list
        push @rooms, $room;
    }

    $player->move($rooms[0]->center->@*);
    return $map;
}
```

Because `$player->move` takes a delta rather than exact coordinates, we'll need
to initialize the player to the top left corner of the map. In our `Engine.pm`

```
field $player = Entity->new(
    x    => 0,
    y    => 0,
    char => '@',
    fg   => '#fff',
    bg   => '#000',
);
```

If we run things now we should get a nice little twisty maze of passages all
alike. Poor Mr. N though is probably still getting dropped into a wall in the
top right corner. Each time you run the game you'll get a new and different
dungeon. Now that we have this pretty little map, next time we'll learn how to
hide it from ourselves so we can explore.

Our code has gotten too big to easily just show it all below. If you'd like to
see the full code listing you can check [here](https://github.com/perigrin/posessive_frogs/tree/part-3)
