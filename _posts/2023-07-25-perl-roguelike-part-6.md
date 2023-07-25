---
layout: post
title: "A Roguelike in Perl Part 6 - Doing (and taking) some damage"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date: 2023-07-25
image: https://live.staticflickr.com/65535/51798060841_3c554e03e3_h.jpg
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

One last minor thing before we get started. We added a param `blocks_moement`
to the Entity class, and had it default to 0. This really seems backwards, like
most of our entities are going to be corporeal and block movement, we should
make that the default and override it for ghosts and what not.

```
field $blocks_movement :param //= 0;
```

Slightly more questionably I'm going to set the default X and Y coordinates to
0 as well.

```
field $x :param //= 0;
field $y :param //= 0;
```

This will allow us to simplify our Entity creation methods a little
as well.

```
package Entities {

    sub goblin() {
        Entity->new(
            char => 'g',
            name => 'goblin',
            fg   => '#41924B',
        )
    }

    sub hobgoblin() {
        Entity->new(
            char => 'h',
            name => 'hobgoblin',
            fg => '#ff6f3c',
        )
    }

    sub player() {
        Entity->new(
            char => '@',
            fg   => '#fff',
            name => 'hero',
        );
    }
```

## Onwards to Part 6

So we left off in [part
5](https://chris.prather.org/perl-roguelike-part-5.html) ready for combat but
hadn't really implemented anything yet. Let's actually get ready to rumble.

First we'll need to sort out some rules for combat. I'm going to be borrowing
from Ben Milton's [Knave](http://questingblog.com/knave/). (Specifically I'm
borrowing from the 1.0 version that you can find on
[archive.org](https://archive.org/details/knave-1.0-en).[^1])

A melee attack is resolved by having the attacker roll a 1d20 and adding their
Strength bonus and the defender rolling a 1d20 and adding their Armor bonus. If
the attacker rolls higher than the defender, they hit. Damage is calculated by
taking the difference of the two rolls.[^2]

So we're going to need some abilities: Strength, Armor, Damage, and Hit Points. 
Let's add a new class to our Entities module.

```
class Abilities {
    use List::Util qw(min);

    field $strength :param;
    field $armor    :param //= 1;
    field $max_hp   :param(hp);
    field $hp = $max_hp;

    method armor()    { $armor }
    method strength() { $strength }
    method hp()       { $hp }
    method change_hp($delta) { $hp += $delta }
}
```

Let's walk through the nuance of this a little. Our `$strength` score must be
passed in. Our `$armor` defaults to a +1 bonus, and we have to
explicitly set our max HP. We have a separate `hp` field to track the _current_
HP for the entity, but that is entirely internal so we're free to alias the
`max_hp` field's constructor arg to just `hp`.

We need to update the Entity class to hold these new Abilities:

```
class Entity {
    field $x :param //= 0;
    field $y :param //= 0;
    field $char: param;
    field $fg :param //= '#fff';
    field $bg :param //= '#000';
    field $name :param //= "<unnamed>";
    field $blocks_movement :param //= 1;
    field $abilities :param;

    method x { $x }
    method y { $y }
    method char { $char }
    method fg { $fg }
    method bg { $bg }
    method blocks_movement { $blocks_movement }
    method stats { $abilities }

    method move($dx, $dy) {
        $x += $dx;
        $y += $dy;
    }
}
```

Let's modify our entity factories to populate these new Abilities:

```
package Entities {
    use List::Util qw(min);
    use Games::Dice qw(roll roll_array);

    sub goblin() {
        Entity->new(
            char => 'g',
            name => 'goblin',
            fg   => '#41924B',
            abilities => Abilities->new(
                strength => -1,
                armor    => 0,
                hp       => roll('1d8-1'),
            )
        );
    }

    sub hobgoblin() {
        Entity->new(
            char => 'h',
            name => 'hobgoblin',
            fg => '#ff6f3c',
            abilities => Abilities->new(
                strength => 1,
                armor    => 1,
                hp       => roll('1d8+1'),
            ),
        );
    }

    sub player() {
        Entity->new(
            char => '@',
            fg   => '#fff',
            name => 'hero',
            abilities => Abilities->new(
                strength => min(roll_array('3d6')),
                armor    => 1,
                hp       => roll('1d8'),  # TODO add CON bonus
            ),
        );
    }
}
```

Rather than write our own dice rolling library let's install one off the CPAN.
Running `cpanm Games::Dice` should work to install the library. We're going to
use it to populate some stats for our entities.

Goblins according to the Monster Manual are slightly weaker than humans, so
they have a -1 strength bonus, their armor is minimal at best so 0 bonus there,
and their hit points are just a little lower than a human.

Hobgoblins on the other hand are about even, if not a little stronger than the
average human, so they get a strength bonus of 1, and an armor bonus of 1, and
do slightly better than normal on hit points.

Our player character is a bit more randomized. Knave says to roll 3 6 sided
dice and take the lowest one for an attribute so that's what we do for our
strength, we take the minimum of 3 d6 dice. We don't have equipment yet so our
armor is the standard value of 1, and our HP is a standard roll, eventually
we'll need to figure out constitution to handel healing and Knave says to add
that to our HP roll so we've left ourselves a note to remember that.

If we run thigns now … nothing has changed. Not surprising because we haven't
actually hooked up the combat rolls. Let us go ahead and do that now.

We start with a new MeleeAttackAction class in our Actions module.

```
class MeleeAttackAction :isa(Action) {
    use Games::Dice qw(roll);

    field $defender :param;

    method perform() {
        my $attacker = $self->entity;
        my $attack_roll = roll('1d20') + $attacker->stats->attack;
        my $defense_roll = roll('1d20') + $defender->stats->armor;

        if ($attack_roll > $defense_roll) {
            $defender->stats->change_hp($defense_roll - $attack_roll)
        }
        return;
    }
}

```

Here we have our `$entity` as our `$attacker` and a take a `$defender` as
required parameter. We have each of them roll and we compare the results. If
the attack roll is higher than the defense roll … the attack succeeds and we
update the defender's hit points with the difference as damage.

Next we update our MovementAction code so that when we bump into an enemy we attack
it instead.

```
if ( my $e = $map->has_entity_at( $x, $y ) ) {
    $combat = MeleeAttackAction->new(
        attacker => $player,
        defender => $e,
    );
    return $combat->perform();
}
```

Now if we run things we should be able to attack our enenmies … but they don't 
actually die. We need to do something about that.

Let's update the MeleeAttackAction a little to check for if we've killed the 
defender.

```
class MeleeAttackAction :isa(Action) {
    use Games::Dice qw(roll);

    field $defender :param;
    field $map :param;

    method perform() {
        my $attacker = $self->entity;
        my $attack_roll = roll('1d20') + $attacker->stats->attack;
        my $defense_roll = roll('1d20') + $defender->stats->armor;

        if ($attack_roll > $defense_roll) {
            $defender->stats->change_hp($defense_roll - $attack_roll)
            if ($defender->stats->hp <= 0) {
                $map->remove_entity($defender);
            }
        }

        return;
    }
}

$defender->stats->change_hp( $defense_roll - $attack_roll );

```

We need to update the call in our MovementAction code to pass in the map:
```
if ( my $e = $map->has_entity_at( $x, $y ) ) {
    $combat = MeleeAttackAction->new(
        map      => $map,
        attacker => $player,
        defender => $e,
    );
    return $combat->perform();
}
```

We'll also need to add the `remove_entity` method to the GameMap:

```
method spawn($e, $x, $y) {
    $e->move($x, $y);
    push @$entities, $e;
}

method remove_entity($e) {
    $entities = [grep { $_ != $e } @$entities];
}
```

Now when we do enough damage to a monster it dies and disappears. These fights
seem kind of one sided though. The monsters aren't really attacking back. Let's
give them some motivation of their own.

First let's write a new class in Entities for all our "mobile" NPCs (aka `Mob`s)

```
class Mob :isa(Entity) {
    method next_action($map) {
        say STDERR $self->name . ' thinks about what to do next.';
    }
}

```

This subclass allows us to have Entities that come up with their own next
actions. We can update the `goblin` and `hobgoblin` functions to use this new
subclass.

```
sub goblin() {
    Mob->new(
        name      => 'goblin',
        char      => 'g',
        fg        => '#41924B',
        abilities => Abilities->new(
            strength => -3,
            armor    => 0,
            hp       => roll('1d8-1'),
        )
    );
}

sub hobgoblin() {
    Mob->new(
        name      => 'hobgoblin',
        char      => 'h',
        fg        => '#ff6f3c',
        abilities => Abilities->new(
            strength => -1,
            armor    => 1,
            hp       => roll('1d8+1'),
        ),
    );
}
```

We should also update the GameMap with a function that runs through 
the Mobs and performs their next actions.

```
method remove_entity($e) {
    $entities = [grep { $_ != $e } @$entities];
}

method update_entities() {
    for my $mob ( grep { $_ isa Mob } @$entities ) {
        my $action = $mob->next_action($self);
        next unless $action;

        $action->perform();
    }
}
```

And we need to trigger this update somewhere, let's do it 
right after we make a move in the Engine.

```
# lets execute the action now
$KEY_MAP{ $event->key }->perform();

# now everyone else gets a turn
$map->update_entities();
```

So far so good. Our mobs now get a turn to perform an action. For maximum
excitement we want them to chase us down and attack us so let's take the pieces
we have and see what we can do:

```
method next_action($map) {
    state $fov = Games::ROT::FOV->new();
    my @visible = $fov->calc_visible_cells_from(
        $self->position->@*,
        8,
        sub ($cell) { $map->tile_at(@$cell)->is_opaque() },
    );

    my @entities = grep { defined } map { $map->has_entity_at( @$_ ) } @visible;
    my $player = first { $_->char eq '@' } @entities;
    return unless $player;

    my @step = Games::ROT::AStar::get_path( $map, $self->position,
        $player->position );

    return unless @step;

    return MovementAction->new(
        entity => $self,
        map    => $map,
        dx     => $step[1]->[0] - $self->x,
        dy     => $step[1]->[1] - $self->y,
    );
}
```

This is a lot but let's break it down. First we use same FOV code that we use
for the player, to grab all the tiles the Mob can see. We turn that into a list
of entities, and try to grab the player character out of it (the player is the
one that looks like an `@`). If the mob can't see a player, don't do anything.

Next we use a new function `Games::ROT::AStar`'s `get_path()` to give us a path
to the player. The A* pathfinding algorithm is very well known and considered a
pretty solid choice, and it happens to be the one that Games::ROT implements.
If you've been following along, you'll need to update `Games::ROT` to the 
latest version: `cpanm https://github.com/perigrin/perl-games-rot.git` should work.

If the mob can't find a path to the player, bail. Otherwise, create a new
`MovementAction` to take the mob one step closer to the player. Remember that
if the next step is _into_ the player our mob automatically attack them using
the exact same combat code used for the player to attack the mobs.

This gives us monsters who chase us and attack us unless we get to them first.

Hmm, you may have noticed that the monsters can move diagonally while the
player cannot. Let's fix that quickly, add the following to the KEY_MAP in the engine:

```
y => MovementAction->new(
    map    => $map,
    entity => $player,
    dx     => -1,
    dy     => -1,
),
u => MovementAction->new(
    map    => $map,
    entity => $player,
    dx     => 1,
    dy     => -1,
),
b => MovementAction->new(
    map    => $map,
    entity => $player,
    dx     => -1,
    dy     => 1,
),
n => MovementAction->new(
    map    => $map,
    entity => $player,
    dx     => 1,
    dy     => 1,
),

```

With that we have a working, albeit exceptionally basic adventure. Next time
we'll explore how to add some polish so we can see what is happening in a bit
more detail.

Last weeks were the last tutorials I had pre-written so things may slow down a
little as I will have to catch back up and try to write two a week as well as
the code to go with them.

Our code has gotten too big to easily just show it all below. If you'd like to
see the full code listing you can check [here](https://github.com/perigrin/posessive_frogs/tree/part-6).

[^1]: Knave 2e is just finishing a [successful kickstarter](https://www.kickstarter.com/projects/questingbeast/knave-rpg-second-edition)

[^2]: Damage isn't calculated this way in Knave but it is in Maze Rats, and in
Cairn and it makes life easier when we don't have equipment for another week.
