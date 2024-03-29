---
layout: post
title: "A Roguelike in Perl Part 1 - Orchestrating Movement in the Dark"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date: 2023-07-04
image: https://live.staticflickr.com/65535/51795827306_55b955aeee_5k.jpg
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [Part 0](https://chris.prather.org/perl-roguelike-part-0.html). If you
remember, the `Engine` class looked like this:

```
class Engine {
    field $height :param;
    field $width  :param;
    field $app = Games::ROT->new(
        screen_width  => $width,
        screen_height => $height,
    );

    ADJUST {
        # start the app
        $app->run( sub { $self->render() } );
    }

    method render() {
        my $x = $width / 2;
        my $y = $height / 2;

        $app->draw($x, $y, 'Hello World', '#fff', '#000');
    }
}

```

First, let's give ourselves a way to quit that doesn't involve killing the
process. Games::ROT allows us to add an event handler, so we'll go ahead and do
that before we start the game loop.

```
ADJUST {
    $app->add_event_handler(
        'keydown' => sub ($event) { exit }
    );
    $app->run( sub { $self->render() } );
}

```

Now if we press a key the game will exit and leave us back in our shell. Next,
we need to replace `Hello World` with an icon for our player.Traditionally the
player is `@`, so let’s add some fields for the player location:

```
class Engine {
    my $height :param;
    my $width  :param;

    field $player_x = $width / 2;
    field $player_y = $height / 2;

    [...]
```

We also need to update the render method to draw the player icon.

```
method render() {
    $app->draw($player_x, $player_y, '@', '#fff', '#000');
}
```

Now if we run things we'll get a nice little `@` in the middle of the terminal,
but that's really not very exciting. We need to get it moving around. Right now
any button we push exits the game, and that simply won't do. Let's fix that by
adjusting our ADJUST block.

```
ADJUST {
    $app->add_event_handler(
        'keydown' => sub ($event) {
            my %KEY_MAP = (
                h => sub { $player_x -= 1 },
                j => sub { $player_y += 1 },
                k => sub { $player_y -= 1 },
                l => sub { $player_x += 1 },
                q => sub { exit }
            );
            $KEY_MAP{$event->key}->();
        }
    );

    $app->run( sub { $self->render() } );
}
```

In our event handler we add a key map to, well, map keys to actions. I'm using
the vim movement pattern here, you're entirely welcome to replace them with
`WASD` or whatever you prefer. Each key is mapped to a little subroutine that,
because it's defined within the scope of the class, has access to the field
variables in that class. We adjust the player's position by one in each
direction. Notice we've also moved quit into its own mapping, rather than an if
statement. At the end we look up the correct key mapping and we just run the
subroutine that's associated with it. Swapping in this code should allow us to
move around.

If you run this code you'll see we're leaving a trail behind us. We need to
clear the screen between each draw.

```
method render() {
    $app->clear();
    $app->draw($player_x, $player_y, '@', '#fff', '#000');
}
```

I'd like to give a shout out to Jeremy for pointing out that I hadn't actually
explained where the `$app->clear()` came in and some leftover typos that had
crept in from earlier versions of this tutorial's code. They're cleaned up now.

That's where we'll leave off for now. Next time we'll refactor things a little
bit and add a non player character, because playing by yourself in an empty
room can get boring.

Here's the full code listing:

```
#!/usr/bin/env perl
use 5.38.0;

use lib qw(lib);
use experimental 'class';

use Games::ROT;

class Engine {
    field $height :param;
    field $width: param;

    field $player_x = $width / 2;
    field $player_y = $height / 2;

    field $app = Games::ROT->new(
        screen_width  => $width,
        screen_height => $height,
    );

    ADJUST {
        $app->add_event_handler(
            'keydown' => sub ($event) {
                my %KEY_MAP = (
                    h => sub { $player_x -= 1 },
                    j => sub { $player_y += 1 },
                    k => sub { $player_y -= 1 },
                    l => sub { $player_x += 1 },
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
        $app->draw($player_x, $player_y, '@', '#fff', '#000');
    }
}

my $engine = Engine->new( width => 80, height => 50 );
```
