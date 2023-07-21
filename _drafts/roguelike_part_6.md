---
layout: post
title: "A Roguelike in Perl Part 6 - Doing (and taking) some damage"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date:
image:
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part 5](https://chris.prather.org/perl-roguelike-part-5.html). If you don't
remember what the code looked like you can refresh yourself with the listing
[here](https://github.com/perigrin/posessive_frogs/tree/part-5).


## Refactoring

When we ended last time we noted that we probably would need to refactor the action. Let's look at the current implementation:

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

It's a simple lexical subroutine, and that was great when there was a single
kind of action, but now that we have potentially several kinds of actions and
actions that potentially spawn new actions, we're gonna need something a bit
more complex. Time for an Action class.

Let's create a new module `Actions.pm` and add the following to it.

```
use 5.38.0;
use warnings;
use experimental 'class';

class Action {
    method perform() { ... }
}

```

The first class we implement `Action` defines a single method `perform()` that
simply dies with `Unimplemented` if you were to execute it. How is this at all
useful you may be asking yourself? Well by itself it wouldn't be, but what it
does is creates an API that we can build upon.

Let's add another class to our Actions file:

```
class MovementAction :isa(Action) { }
```

The `MovementAction` does something we haven't needed to do yet in this tutorial,
it inherits from the `Action` class via the `:isa(Action)` attribute in the
class declaration. This means that anywhere someone has a `MovementAction` object
they can call methods declared in `Action` and if they're not _also_ declared
in `MovementAction` then it will look up the method in `Action` class and call it
instead, on the same object.

Inheritance is tricky and difficult to use without running into problems in
places, normally I would advise against using it if possible. The current
implementation of the class feature though doesn't have a lot of other options
available yet and this is as good as any to demonstrate how to use it, and to
talk about some of it's pitfalls.

First lets expand our `MovementAction` class a bit:

```
class MovementAction :isa(Action) {
    field $dx :param = 0;
    field $dy :param = 0;

    field $map :param;

    method perform() {
        my ( $x, $y ) = ( $player->x + $dx, $player->y + $dy );
        return unless $map->is_in_bounds( $x, $y );
        return unless $map->tile_at( $x, $y )->is_walkable;
        if ( my $e = $map->has_entity_at( $x, $y ) ) {
            return if $e->blocks_movement;
        }
        $player->move( $dx, $dy );
    }
}
```

The `perform()` method should be familiar, it's a direct copy of the
`movement_action` function from our `Engine` class, except instead of the `$dx`
and `$dy` parameters being provided to the function they're provided as fields
on the `MovementAction` class. If we were to run this like it is though we
would have a problem, `$player` isn't defined.

Every Action should have an entity that performs that action so we can easily
just modify the Action class like so

```
class Action {
    field $entity :param;
    method perform() { ... }
}
```

But with the current implementation of `feature class` you cannot inherit state
from your parent. There are [good
reasons](https://github.com/Ovid/Cor/issues/61#issuecomment-1176095406) for
this that basically are summarized with "it can easily lead to nasty bugs that
look like they're what you want but in practice break in ways you don't
expect." So we will need to provide a way for the `MovementAction` to get
access to that data.

```
class Action {
    field $entity :param;

    method entity {
        die 'protected method' unless caller()->isa(__PACKAGE__);
        return $entity
    }

    method perform() { ... }
}
```

Unfortunately we don't have a way to explicitly limit methods to only be
visible by our sub-classes. This is a feature that is [currently under
discussion](https://github.com/Ovid/Cor/issues/61) so we're gonna have to fake
it. We write a normal reader method like we have a dozen times before but we
add a check this time to make sure that the caller inherits from the current
package: `die 'protected method' unless caller()->isa(__PACKAGE__);` The new
`feature class` doesn't replace the existing perl Object System it has just
extended it in some select ways to make it easier and more natural based on
developments that have happened since 1995.

Now in our `MovementAction` class's `perform()`  method we can add one line:
```
method perform() {
    my $player = $self->entity();
    my ( $x, $y ) = ( $player->x + $dx, $player->y + $dy );
    return unless $map->is_in_bounds( $x, $y );
    return unless $map->tile_at( $x, $y )->is_walkable;
    if ( my $e = $map->has_entity_at( $x, $y ) ) {
        return if $e->blocks_movement;
    }
    $player->move( $dx, $dy );
}
```

Before we forget, we'll also need a QuitAction class. Thankfully this one is pretty simple.

```
class QuitAction :isa(Action) {
    method perform() { exit }
}
```

Our code should now be ready to be hooked into the engine, before sure to load the new Actions module.

```
use Entities;
use Actions;
```

Then update the `ADJUST` block to use the MovementAction class:
```
ADJUST {
    $app->add_event_handler(
        'keydown' => sub ($event) {
            my %KEY_MAP = (
                h => MovementAction->new(
                    map    => $map,
                    entity => $player,
                    dx     => -1
                ),
                j => MovementAction->new(
                    map    => $map,
                    entity => $player,
                    dy     => 1
                ),
                k => MovementAction->new(
                    map    => $map,
                    entity => $player,
                    dy     => -1
                ),
                l => MovementAction->new(
                    map    => $map,
                    entity => $player,
                    dx     => 1
                ),
                q => QuitAction->new(
                    entity => $player
                ),
            );

            # lets execute the action now
            $KEY_MAP{ $event->key }->perform();
        }
    );
    $app->run( sub { $self->render() } );
}
```
## Onwards to Part 6

So we left off in [part
5](https://chris.prather.org/perl-roguelike-part-5.html) ready for combat but
hadn't really implemented anything yet. Let's actually get ready to rumble.



Our code has gotten too big to easily just show it all below. If you'd like to
see the full code listing you can check [here](https://github.com/perigrin/posessive_frogs/tree/part-6).

