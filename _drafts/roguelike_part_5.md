---
layout: post
title: "A Roguelike in Perl Part 5 - Knowing where your enemies are, and how to kick them"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image:
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part-4](). If you don't
remember what the code looked like you can refresh yourself with the listing
[here](https://github.com/perigrin/posessive_frogs/tree/part-5).

We have a darkened dungeon you can explore and while that can be exciting, it
lacks a certain element of danger. Where are the monsters that are supposed to
lurk around every corner?

## Spawning Mobs

Right now we're spawning all of our Entities in the Engine class. That's easy
when you only have a player and Mr. N, and poor Mr. N is being spawned inside a
wall in the corner of the map!

Let's start with modifying the GameMap class to have access to the Entities
directly so we can know where on the floor (and eventually what floor) our
entities are on.

```
use warnings;
use experimental 'class';

use Entities;
```

We import the Entities class into the GameMap. Next we'll add a new field to
the GameMap class itself.

```
field $width    :param;
field $height   :param;
field $entities :param;
```

We have to use a scalar `$entities` field here because in the current version
of the class feature, the `:param` attribute doesn't work on non-scalar fields.
This is part of why it's still experimental. Future versions should have a way
to work around this limitation or at least advice on how to live with it.

Next let's remove them from the Engine. We'll delete the `field @npcs`
declaration so the code now looks like:

```
 field $player = Entity->new(
       x    => 0,
       y    => 0,
       char => '@',
       fg   => '#fff',
       bg   => '#000',
    );

    field $app = Games::ROT->new(
        screen_width  => $width,
        screen_height => $height,
    );
```

Notice for now we've left the Player object. We'll need to handle it a little
differently. But next we'll modify the Engine's `render` method to not render
the entities because that'll belong to the GameMap.

```
method render() {
    update_fov($map, $player);
    $app->clear();
    $map->render($app);
}
```

We can in fact just move that code into the GameMap's render method.

```
method render($term) {
    $self->for_each_tile(sub ($tile, $x, $y) {
        $term->draw($x, $y, $tile->char, $tile->fg, $tile->bg) if $tile->seen;
    });
    for my $e ($entities->@*) {
        my $tile = $self->tile_at($e-x, $e->y);
        $term->draw($e->x, $e->y, $e->char, $e->fg, $e->bg) if $tile->visible;
    }
}
```

Notice we added a line. This fixes a little bug we had from before where we
could still see Mr. N despite him being stuck in a wall. Now we only display
entities on visible tiles.

Finally we need to alter the place in the SimpleDungeonGenerator where we create the
 GameMap object and pass in our player.

```
method generate_dungeon() {
    my $map = GameMap->new(width  => $width, height => $height, entities => [$player]);
    ...
}
```

Now if we run the game we'll not see any changes, except Mr. N is gone.

So let's move on to actually putting monsters in our mazes. For the purposes of
this tutorial we'll keep it simple: each room will have a random number of
monsters between 0 and n (n=2 for now). We'll have an 80% chance to spawn a
Goblin and a 20% chance to spawn a Hobgoblin (the bigger badder version).

In our SimpleDungeonGenerator let's add a new field to track:

```
field $width    :param;
field $height   :param;
field $rooms    :param(room_count);
field $max_monsters_per_room :param;
field $min_size :param(min_room_size);
field $max_size :param(max_room_size);
field $player   :param;
```

And back in the Engine class where we create the generator object we need to
pass that value in.

```
field $map = SimpleDungeonGenerator->new(
    room_count            => 30,
    min_room_size         => 6,
    max_room_size         => 10,
    max_monsters_per_room => 2,
    width                 => $width,
    height                => $height,
    player                => $player,
)->generate_dungeon();
```

Now that we know how many we want (max), how do we go about setting them loose
on unsuspecting adventurers? Inside our `generate_dungeon()` method let's add a
call to a new lexical function `place_entities()`.

```
# tunnel between the previous room and this one
tunnel_between($map, $rooms[-1]->center, $room->center) if @rooms;

# add some monsters
place_entities($room, $map, $max_monsters_per_room);

# add the room to the list
push @rooms, $room;
```

and now we just need to write `place_entities()`

```
my sub place_entities($room, $map, $max) {
    my $num = int rand($max + 1); # need one more than we want cause 0 is a choice
    for my $i (1..$num) {
        my ($x, $y) = $room->random_point()->@*;
        if (! $map->has_entity_at($x, $y)) {
            # TODO add a goblin/hobgoblin to the map in a 4:1 ratio
        }
    }
}
````

That gets us most of the way there, we start by grabbing a random number
between 0 and our max, then at a random point in the room that doesn't already
have an entity in it we place a new entity. We still need to figure out what it
actually means to place a goblin or a hobgoblin, or even what it means to _be_
a goblin or a hobgoblin. While we think about that let's look at those utility
methods we just snuck in.

The `random_point` method in the RectangularRoom class looks like:

```
method random_point() {
    my $x = $x1 + rand($width) + 1;
    my $y = $y1 + rand($height) + 1;
    return [$x, $y]
}

method center() {

```

and the `has_entity_at` method in the GameMap class looks like

```
method has_entity_at($x, $y) {
    first { $_->x == $x && $_->y == $y } $entities->@*
}

method is_in_bounds($x, $y) {
```

and we will need to import List::Util's `first` function at the start of
GameMap

```
class GameMap {
    use List::Util qw(first);
```

Now about putting in the new entities. We _could_ just put the appropriate
calls to Entity in there and be done with it. With only two kinds of monsters
that might satisfy, but when we start scaling up eventually that's going to
become problematic. Not to mention what happens if we want a
LessSimpleDungeonGenerator or a MildlyComplicatedDungeonGenerator or a
LevelThatIsNotADungeonButANiceForestGladeGenerator. Do we copy over the
instance creations for each of these and duplicate them? That's gonna lead to
typos where the goblins in the glade kick way more ass than the goblins in the
complicated dungeon.

There's a better way. Let's start by updating our Entity class:

```
field $fg :param //= '#fff';
field $bg :param //= '#000';
field $name :param //= "<unnamed>";
field $blocks_movement :param = 0;
```

Now our entities have a name and we can tell if they should block movement or
not. Let's also add a new package of helper functions:

```
package Entities {
    sub goblin() {
        Entity->new(
            x => 0,
            y => 0,
            char => 'g',
            name => 'goblin',
            fg   => '#41924B',
            blocks_movement => 1
        )
    }

    sub hobgoblin() {
        Entity->new(
            x => 0,
            y => 0,
            char => 'h',
            name => 'hobgoblin',
            fg => '#ff6f3c',
            blocks_movement => 1
        )
    }

    sub player() {
        Entity->new(
            x    => 0,
            y    => 0,
            char => '@',
            fg   => '#fff',
            name => 'hero',
            blocks_movement => 1,
        );
    }
}
```

The Entities is a factory package that provides us with functions to create the
different kinds of entities. Let's go back and put these into the
`place_entities()` function.

```
my sub place_entities($room, $map, $max) {
    my $num = int rand($max + 1); # need one more than we want cause 0 is a choice
    for my $i (1..$num) {
        my ($x, $y) = $room->random_point()->@*;
        if (! $map->has_entity_at($x, $y)) {
            my $e = rand() < 0.8 ? Entities::goblin() : Entities::hobgoblin();
            $map->spawn($e, $x, $y);
        }
    }
}
```

And  last but not least the spawn method on the GameMap

```
method spawn($e, $x, $y) {
    $e->move($x, $y);
    push @$entities, $e;
}
```

Because our newly spawned entity should default to 0,0 we can just use `move`
here. If that weren't true we'd have to either have a method to teleport the
entity to a specific x,y coordinate or to reset the coordinates to 0,0.

We can actually update the Engine to use the `Entities::player()` function as
well.

```
field $player = Entities::player();

field $app = Games::ROT->new(
```

And with that the dungeon should now populate with monsters if we run it again.
They're not very imposting monsters though, in fact they let us walk all over
them.

In the Engine class we need to add another check to our `movement_action`:

```
my sub movement_action($dx, $dy) {
    my ($x, $y) = ($player->x + $dx, $player->y + $dy);
    return unless $map->is_in_bounds($x, $y);
    return unless $map->tile_at($x, $y)->is_walkable;
    if (my $e = $map->has_entity_at($x, $y)) {
        if ($e->blocks_movement) {
            say sprintf('The %s gives the %s a swift kick.', $player->name, $e->name);
            return;
        }
    }
    $player->move($dx,$dy);
}
```

The `$map->has_entity_at` happens to return the first entity it finds at a
given location, so we check to see if we can walk through that entity. This
could be problematic if we add ghosts but it's good enough for now.

Finally we'll need to update the Entity class to have a `blocks_movement`
method exposing the `blocks_movment` field.

```
method blocks_movement { $blocks_movement }

method move($dx, $dy) {
```

This article is starting to get a little long and we've accomplished a lot. We
will save the finer nuances of combat actions for next time, we'll start with a
refactor of the action system to be more extensible.

Our code has gotten too big to easily just show it all below. If you'd like to
see the full code listing you can check [here](https://github.com/perigrin/posessive_frogs/tree/part-5).
