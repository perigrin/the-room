---
title: "Part 4 - Field of View"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image:
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part-3](). If you don't
remember what the code looked like go back and refresh yourself.

We have a dungeon that we can walk around in but it's not really _exploring_ if
we can just see everything from the get go. Most have a certain fog of war, and
limit what you can see to a Field of View. In this post we're going to lay the
ground work for that.

Let's start with some small alterations to our Tile class to handle a couple
new cases.

```
class Tile {
    field $walkable :param;
    field $opaque :param;
    field $char :param //= ' ';
    field $light_fg :param //= '#fff';
    field $light_bg :param //= '#000';
    field $dark_fg  :param //= '#333';
    field $dark_bg  :param //= '#000';

    field $seen = 0;

    method is_walkable() { $walkable }
    method is_opaque { $opaque }
    method char() { $char }

    method fg($visible=undef) {
        $visible ? $light_fg : $dark_fg;
    }
    method bg($visible=undef) {
        $visible ? $light_bg : $dark_bg;
    }

    method seen($new=undef) {
        if (defined $new) { $seen = $new }
        return $seen;
    }
}
```

Let's break down these changes. We added an `$opaque` field to determine if we
can see through a tile or not, along with the `is_opaque` method. Then we broke
our `$fg` and `$bg` fields out into ones for light and ones for dark. Next we
update the `fg` and `bg` methods to take an attribute for if the tile is
_currently_ visible or not. Next we add a field for if we've `$seen` this tile
before and a new accessor method to get and potentially set the seen status. If
a tile isn't seen we simply won't render it, so we'll take care of that in the
map rendering method.

```
method render($term, $x_slice, $y_slice) {
	for my $y (0..$height) {
		for my $x (0..$width) {
			my $tile = $self->tile_at($x, $y);
			$term->draw($x, $y, $tile->char, $tile->fg, $tile->bg) if $tile->seen;
		}
	}
}
```

The biggest single change comes in the engine class. We need to create a little
helper function. Again we're going to use a lexical subroutine to keep this
scoped to the Engine class.

```
my sub update_fov($map, $player) {
	state $fov = Games::ROT::FOV->new();

	$map->for_each_tile(sub ($tile, @){ $tile->visible(0) });

	my @cells = $fov->calc_visible_cells_from(
		$player->x,
		$player->y,
		8,
		sub ($cell) { $map->tile_at(@$cell)->is_opaque() }
	);
	for my $cell (@cells) {
		my $tile = $map->tile_at(@$cell);
		$tile->visible(1);
	}
}
```

First thing we do is create an instance of the `Games::ROT::FOV` class which is
a utility class that calculates a Field of View. We make this a `state`
variable just to keep ourselves from creating it over and over again.

Next we'll call a utility method that we haven't written yet that applies a
function to every tile in the map. The function just resets the visibility of
the tiles to a default.

Next we have the FOV calculator figure out what cells the player can see, we're
using a default radius of 8, but I've thought about making this an attribute of
the player. The last argument to `calc_visible_cells_from` is a function that
determines if a cell is opaque or not.

Once we have the list of cells that are visible to the player, we just loop
over those and set their visibility to true. Finall we update our Engine's
render method to call the `update_fov()` function before we render anything.

```
method render() {
	update_fov($player);
	$app->clear();
	$map->render($app);
	for my $e (@npcs, $player) {
		$app->draw($e->x, $e->y, $e->char, $e->fg, $e->bg);
	}
}
```

Wait! What about that utility method you say? Let's take a look at it, it'll be
in our GameMap class.

```
method for_each_tile($action) {
	for my $y (0..$height) {
		for my $x (0..$width) {
			my $tile = $self->tile_at($x, $y);
			$action->($tile, $x, $y);
		}
	}
}
```

It's basically the Map's render method with the guts replaced with executing
the function we pass in. In fact it's so similar to the render method we could
just update that to all `for_each_tile` and pass in the right callback.

```
method render($term) {
	$self->for_each_tile(sub ($tile, $x, $y) {
		$term->draw($x, $y, $tile->char, $tile->fg, $tile->bg) if $tile->seen;
	});
}
```

All in all this wasn't very complicated, well unless you had to write the
Games::ROT::FOV but thankfully someone has taken care of that for us. We now
have dungeons to actually _explore_ but it seems a little empty and poor Mr. N
is still stuck in the corner. We'll start fixing those problems next.

If you'd like to see the full code listing you can check
[here](https://github.com/perigrin/posessive_frogs/tree/part-4).
