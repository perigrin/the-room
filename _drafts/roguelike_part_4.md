---
title: "Part 4 - Field of View"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image:
---

## Introduction

This post is part of a series of blog posts following the [roguelike
tutorial](https://www.rogueliketutorials.com/) to demonstrate the new `class`
feature in Perl 5.38.0.

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
    field $transparent :param;
    field $char :param //= ' ';
    field $light_fg :param //= '#fff';
    field $light_bg :param //= '#000';
    field $dark_fg  :param //= '#333';
    field $dark_bg  :param //= '#000';

    field $seen = 0;

    method is_walkable() { $walkable }
    method is_transparent { $transparent }
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

Let's break down these changes. First we broke our `$fg` and `$bg` fields out
into ones for light and ones for dark. Next we update the `fg` and `bg` methods
to take an attribute for if the tile is _currently_ visible or not. Next we add
a field for if we've `$seen` this tile before and a new accessor method to get
and potentially set the seen status. If a tile isn't seen we simply won't
render it, so we'll take care of that in the rendering methods in a minute.


