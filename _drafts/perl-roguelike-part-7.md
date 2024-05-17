---
layout: post
title: "A Roguelike in Perl Part 7 - Some Simple UX"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date: 2024-04-22
image: https://live.staticflickr.com/65535/52933412971_af0b2aba64_k.jpg
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part 6](https://chris.prather.org/perl-roguelike-part-6.html). If you don't
remember what the code looked like you can refresh yourself with the listing
[here](https://github.com/perigrin/posessive_frogs/tree/part-6).

## The purest and most thoughtful minds are those which love color the most.

So it's been a while since our last installment and we're just going
to move foward and ignore any major refactors we might have done since the last
installment (I checked. There weren't any, but still.)

This time we're going to be laying out a minimal user interface so that as we
play we have some idea of what is going on. First up let's start by building a
file that contains some of our common colors.

```perl
use 5.38.0;
use warnings;
use experimental 'class';

package Colors {
    use constant White => '#fff';
    use constant Black => '#000';

    use constant DefaultEntityFG => White;
    use constant DefaultEntityBG => Black;
    use constant Goblin          => '#41924B';
    use constant Hobgoblin       => '#ff6f3c';
    use constant Hero            => White;

    use constant DefaultLightTileFG => White;
    use constant DefaultLightTileBG => Black;
    use constant DefaultDarkTileFG  => '#666';
    use constant DefaultDarkTileBG  => Black;

}
```

And we update our code to use these new constants, like in `Entity.pm`, in our
Entity class we swap out:

```perl
field $fg :param //= '#fff';
field $bg :param //= '#000';
```
for

```perl
field $fg :param //= Colors::DefaultEntityFG;
field $bg :param //= Colors::DefaultEntityBG;
```

We can also set the Goblin color from `#41924B` to `Colors::Goblin` and the
Hobgoblin color from `#ff6f3c` to `Colors::Hobgoblin`, and the player color
from `#fff` to `Colors::Hero`. Also reset the colors in the `GameMap` class to
their new constants as well.

Last time we implemented a basic HP tracking system for when we take (or deal)
damage. Let's display that information so the player can actually see it. We'll
create a bar that gradually decreases as the player loses HP. This will also
help show the player how much HP is remaining. We'll create a method called
`render_bar` in our `Engine` class.

```perl
method render_bar ( $current, $max, $total ) {
	my $width = int( ( $current / $max ) * $total );

	$app->draw_rect( 8, 45, $total, 0, ' ', Colors::BarEmpty, Colors::BarEmpty );
	if ($width) {
		$app->draw_rect( 8, 45, $width, 0, ' ', Colors::BarFilled, Colors::BarFilled );
	}
	$app->puts( 0, 45, "hp: $current/$max" );
}
```

This code requires the latest version of `Games::ROT`. If you've been following
along, you'll need to update `Games::ROT` to the latest version: `cpanm
https://github.com/perigrin/perl-games-rot.git` should work.

What this will do is draw a new rectangle starting at `(8,45)` and render it in
the BarEmpty color. Let's add that color to our `Colors` package:

```perl
package Colors {
	...
	use constant BarEmpty  => '#411';
}
```

Then if we have data for a filled bar (calculated as a percentage of the width
of the default bar) we draw it starting at the same place but with the reduced
width. So let's add the `BarFilled` color as well.

```perl
package Colors {
	...

	use constant BarFilled => '#060';
	use constant BarEmpty  => '#411';
}
```

Then we write out some text with the actual numbers. Note that we start this
text at `(0,45)` this should fit in the gap we set up before by starting the
bar at `(8,45)`.

We can hook this code up by adding a call to the Engine's `render` method:

```perl
method render() {
	update_fov( $map, $player );
	$app->clear();
	$map->render($app);
	$self->render_bar( $player->stats->hp, $player->stats->max_hp, 20 );
}
```

If we run the game now we might notice that the bar overwrites some of the game
map. We can adjust the game map to take this into account where we create it at
the top of the Engine class.

```perl
field $map = SimpleDungeonGenerator->new(
	room_count            => 30,
	min_room_size         => 6,
	max_room_size         => 10,
	max_monsters_per_room => 2,
	width                 => $width,
	height                => $height - 6, # take into account our UX
	player                => $player,
)->generate_dungeon();
````

Another annoying problem is that we have no way to message the player what
actually transpired. We can now know we lost health but not why. Let's add a
message log as well. We can start with a new `MessageLog.pm` file with a
basic implementation like this:

```perl
use 5.38.0;
use experimental 'class';

use Colors;
use Text::Wrap;

class Message {
    field $text : param;
    field $fg : param;
    field $count = 1;

    method eq ($msg) {
        if ( $msg isa Message ) {
            return $msg->eq($text);
        }
        return "$msg" eq $text;    # pray it stringifies and comp then
    }

    method inc() {
        return $count += 1;
    }

    method color() { $fg }

    method full_text() {
        if ( $count > 1 ) {
            return "$text (x$count)";
        }
        return $text;
    }

}

class MessageLog {

    field @messages = ();

    sub instance($) { state $log = __PACKAGE__->new() }

    method add_message ( $msg, $fg = Colors::White, $stack = !!1 ) {
        if ( $stack && @messages && $messages[-1]->eq($msg) ) {
            $messages[-1]->inc();
        }
        else {
            push @messages, Message->new( text => $msg, fg => $fg );
        }
    }

    my $lines = sub ( $text, $width = 80 ) {
        local $Text::Wrap::columns  = $width;
        local $text::Wrap::unexpand = 0;
        split '\n', Text::Wrap::wrap( '', '', $text );
    };

    method render ( $term, $x, $y, $w, $h ) {
        my $off = $h - 1;
        for my $msg ( reverse @messages ) {
            for my $line ( reverse $lines->( $msg->full_text, $w ) ) {
                $term->puts( $x, $y + $off--, $line, $msg->color );
                return if $off < 0;
            }
        }
    }
}
```

Let's break break break it down. We have a `Message` class that holds a
message, an color, and a count. The count is used to stack messages of the same
type. We also have a `MessageLog` class that holds a list of messages and can
render them to the screen. Our `render` method will render the messages in
reverse order, starting from the bottom of the screen. This prevents the
messages from overwriting each other. We also wrap the text to fit the width of
our message area.

We can now add a message log to our `Engine` class:

```perl
field $message_log = MessageLog->instance();
```

We're using the `instance` method to create a singleton instance of the
`MessageLog` class so we can access the same message log from anywhere in our
code. This will be useful in a second.

Right now let's add a call to the Log's `render` method in our `Engine`'s `render` method:

```perl
method render() {
    update_fov($map, $player);
    $app->clear();
    $map->render($app);
    $self->render_bar( $player->stats->hp, $player->stats->max_hp, 20 );
    $log->render( $app, 40, 45, 40, 5 );
}
```

This will render the message log starting at `(40,45)` in a 40x5 row area. Now let's add a message to the log when the game starts up, in the `ADJUST` block of the engine add the following line just before the call to `run`:

```perl
$log->add_message("Welcome adventuer, to yet another dungeon!", Colors::WelcomeText);
```

If we run the game now we should see the message log with the welcome message. We also want to add a message to the log when the player takes or deals damage. In the `Action` class we start by adding a reference to the message log:

```perl
field $log = MessageLog->instance(); # grab an instance of the message log

method log ($msg, $color) {
    die 'protected method' unless caller()->isa(__PACKAGE__);
    $log->add_message($msg, $color);
}
```

Now all of our `Action` subclasses can call the `log` method to add messages to the logs. Let's add a message to the `MeleeAttackAction` class. Before where it said:

```perl
    $defender->stats->change_hp( $defense - $attack_roll );
```

Let's replace that with:

```perl
    my $damage = $defense - $attack_roll;
    $self->log(
        sprintf('%s attacks %s for %d damage', $attacker->name, $defender->name, $damage),
        Colors::Attack
    );
    $defender->stats->change_hp($damage);
```


 We can also add a message when the player dies. Add the following code to the `remove_entity` method in the `GameMap` class:

```perl
    if ($e->car eq '@') {
        $log->add_message('You died. Game over.', Colors::GameOver);
        exit;
    }
```

And we need to add these new colors to our `Colors` package:

```perl
use constant WelcomeText => '#2AF';
use constant GameOver    => '#F00';
use constant Attack      => '#C00';
```

Now if we run the game we get a little more feedback when we attack or get attacked, and when we die.

Next time we'll add a simple inventory system to the game and some items to pick up and use.
