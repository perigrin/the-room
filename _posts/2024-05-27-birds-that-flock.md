---
layout: post
title: "Birds that Flock Together"
author: "Chris Prather"
tags: perl, dev, ecs, game-dev, corinna
date: 2024-05-27
image: https://images.unsplash.com/photo-1555511920-5d0d9834f549?q=80&w=2674&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

So I've been prepping for my [class on Game Development](https://tprc2024.sched.com/event/1d68A) and one of
the things I want to introduce to people is the concept of an ECS (Entity
Component System). I mentioned [last time](https://chris.prather.org/the-wasteland.html)
about how I needed to build a new set of bindings for a graphics library, and
as part of that I wanted to port a little script I'd done implementing the
[boids](https://en.wikipedia.org/wiki/Boids) algorithm.

Let's walk through the code and see how it works. First we have the ECS class
itself. This is a simple class that holds a list of entities and components and
provides a way to query for entities that have a specific set of components.

Component are stored in two places, once in an array of hashes for the
entities, the index being the entity id. The second is a hash of arrays, where
the value is an array of entity ids that have that component. Initially the key
is the component name, but later we will get a little more complicated to
implement something called an archetype.

```perl
class ECS {
    field $systems : param;

    field @entities   = ();
    field %components = ();

    method entity_count { return scalar grep defined, @entities }

    method add_entity() {
        push @entities, {};
        return $#entities;
    }

    method add_component ( $entity, $component ) {
        $entities[$entity]{ ref $component } = $component;
        push $components{ ref $component }->@*, $entity;
    }

    my sub intersection (@lists) {
        my %u = ();
        return grep { $u{$_}++ } map { $_->@* } @lists;
    }

    method entities_with (@components) {

        # if we have no components, return all entities
        if ( $components{"@components"} ) {
            return grep defined, @entities[ $components{"@components"}->@* ];
        }

        # otherwise make a new archtype
        $components{"@components"} =
          [ intersection( @components{@components} ) ];
        return grep defined, @entities[ $components{"@components"}->@* ];
    }

    method update () { $_->update($self) for $systems->@* }
}

```

You can see the little bit of complexity here with the `entities_with` method
where we will automatically create a new archetype if it doesn't already exist.
An archetype is a group of entities that all have the same set of components.
This is a common optimization in ECS systems to avoid having to check every
entity for every system.

Unfortunately, it doesn't help us much with boids, as we'll see in a bit.

Next we have the components themselves. These are simple classes that hold data
and maybe do some very simple processing on that data. Profiling with NYTProf
showed that the more time we spent in the components the more method
dispatches, the slower the system. So we try to keep the components as simple
as possible.

```perl
class Position {
    field $x : param;
    field $y : param;
    field $vx = rand();
    field $vy = rand();

    method location { return ( $x,  $y ) }
    method velocity { return ( $vx, $vy ) }

    method vx ($dvx) { $vx = $dvx }
    method vy ($dvy) { $vy = $dvy }

    method update_location() {
        $x += $vx;
        $y += $vy;
    }

}

class Vision {
    field $range : param : reader = 30;

}

class Proximity {
    field $distance : param : reader = 5;
}

```

Notice also we're using v5.40.0's new feature to the `class` syntax `:reader`
which automatically creates a read-only accessor for a field.

Next we have the systems. These are classes that implement the core logic. For
documenting purposes I've added a little Abstract Base Class `System` that
doesn't do anything but document the API that all systems must implement. In
Moose we'd use `Moose::Role` for this, but we don't have roles in core ... yet.

```perl
class System {
    method update ($ecs) { ... }
}
```

The first system is the `Avoidance` system. This system looks at all the
entities with a `Position` and `Proximity` component and if they are too close
to each other it moves them away from each other. This is partly where our
little ECS falls down. We have to iterate over all the entities with a
`Position` component to find the ones we want to avoid. If I had done a bit
more work up front with some Data Oriented Design I would have thought about
how we were querying the entities, and I could have optimized storing the
entities in a way that made this faster. Paying attention to the locality of
your data isn't just something for low level programming.

```perl
class Avoidance : isa(System) {
    field $avoidance_factor : param = 1.5;

    method update ($ecs) {
        my @others =
          map { entity => $_, location => [ $_->{'Position'}->location ], },
          $ecs->entities_with('Position');

        for my $entity ($ecs->entities_with( 'Position', 'Proximity' )) {
            my $cdx = 0;
            my $cdy = 0;
            my ( $pos, $prox ) = $entity->@{ 'Position', 'Proximity' };
            my ( $x, $y )      = $pos->location();
            my ( $vx, $vy )    = $pos->velocity();
            my $sq_distance = $prox->distance**2;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                unless ( $dx**2 + $dy**2 > $sq_distance ) {
                    $cdx += $dx;
                    $cdy += $dy;
                }
            }
            $pos->vx( $vx + $cdx / $avoidance_factor );
            $pos->vy( $vy + $cdy / $avoidance_factor );
        }
    }
}

```

Next we have the `Alignment` system. This system looks at all the entities with
a `Vision` component and then checks the `Position` of all the other entities
to see if they are within the range of the `Vision`. If they are, it averages
the velocity of all the entities in the vision range and then adjusts the
velocity of the current entity to match that average.

Another optimization you might notice is that we're querying the "other"
entities up front and grabbing just the data we need. Again if I had thought
about this up front I could have stored the entities in a way that made this
faster.

```perl
class Alignment : isa(System) {
    field $matching_factor : param = 0.08;

    method update ($ecs) {
        my @others =
          map {
            entity     => $_,
              location => [ $_->{'Position'}->location ],
              velocity => [ $_->{'Position'}->velocity ],
          }, $ecs->entities_with('Position');

        for my $entity ( $ecs->entities_with( 'Position', 'Vision' ) ) {
            # create some variables to store the average position, velocities and the number of boids in the vicinity
            my ($xpos_avg,$ypos_avg, $xvel_avg, $yvel_avg, $neighboring_boids);

            my ( $pos, $vis ) = $entity->@{ 'Position', 'Vision' };
            my ( $x,   $y )   = $pos->location();
            my $range = $vis->range;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                if ( abs($dx) < $range && abs($dy) < $range ) {
                    my ( $vx, $vy ) = $other->{velocity}->@*;
                    $xvel_avg          += $vx;
                    $yvel_avg          += $vy;
                    $neighboring_boids += 1;
                }
            }

            if ($neighboring_boids) {
                $xvel_avg = $xvel_avg / $neighboring_boids;
                $yvel_avg = $yvel_avg / $neighboring_boids;

                my ( $vx, $vy ) = $pos->velocity();
                $pos->vx( $vx + ( $xvel_avg - $vx ) * $matching_factor );
                $pos->vy( $vy + ( $yvel_avg - $vy ) * $matching_factor );
            }
        }
    }
}
```

The last of our business logic systems is the `Cohesion` system. Like the
`Alignment` system it looks at all the entities with a `Vision` component and
then checks the `Position` of all the other entities to see if they are within
the range of the `Vision`. If they are, it averages the position of all the
entities in the vision range and then adjusts the vector of the current entity
to move towards the center of the flock.

```perl
class Cohesion : isa(System) {
    field $centering_factor = 0.0002;

    method update ($ecs) {
        my @others =
          map { entity => $_, location => [ $_->{'Position'}->location ], },
          $ecs->entities_with('Position');

        for my $entity ( $ecs->entities_with( 'Position', 'Vision' ) ) {
            # create some variables to store the average velocities and the number of boids in the vicinity
            my ($xpos_avg, $ypos_avg, $neighboring_boids);

            my ( $pos, $vis ) = $entity->@{ 'Position', 'Vision' };
            my ( $x,   $y )   = $pos->location();
            my $range = $vis->range;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                if ( abs($dx) < $range && abs($dy) < $range ) {
                    $xpos_avg          += $other_x;
                    $ypos_avg          += $other_y;
                    $neighboring_boids += 1;
                }
            }
            if ($neighboring_boids) {
                $xpos_avg = $xpos_avg / $neighboring_boids;
                $ypos_avg = $ypos_avg / $neighboring_boids;
                my ( $vx, $vy ) = $pos->velocity();

                $pos->vx( $vx + ( $xpos_avg - $x ) * $centering_factor );
                $pos->vy( $vy + ( $ypos_avg - $y ) * $centering_factor );

            }

        }
    }
}
```

Next we have a couple utility systems. The `ScreenEdge` system keeps the boids
from flying off the screen by adjusting their velocity when they get too close
to the edge. The `SpeedLimits` system keeps the boids from flying too fast or
too slow by adjusting their velocity when they exceed a maximum speed, or fall
below a minimum speed.

```perl
class ScreenEdge : isa(System) {
    field $width : param;
    field $height : param;

    field $turn_factor : param = 0.02;
    field $margin : param      = 90;

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            my $pos = $entity->{'Position'};
            my ( $x,  $y )  = $pos->location();
            my ( $vx, $vy ) = $pos->velocity();

            if ( $x < $margin )           { $pos->vx( $vx + $turn_factor ) }
            if ( $y < $margin )           { $pos->vy( $vy + $turn_factor ) }
            if ( $x > $width - $margin )  { $pos->vx( $vx - $turn_factor ) }
            if ( $y > $height - $margin ) { $pos->vy( $vy - $turn_factor ) }
        }
    }
}

class SpeedLimits : isa(System) {
    field $min_speed = 2.0;
    field $max_speed = 4.0;

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            my $pos = $entity->{'Position'};

            my ( $vx, $vy ) = $pos->velocity();
            my $speed = sqrt( $vx * $vx + $vy * $vy ) || $min_speed;
            if ( $speed < $min_speed ) {
                $pos->vx( ( $vx / $speed ) * $min_speed );
                $pos->vy( ( $vy / $speed ) * $min_speed );
            }

            if ( $speed > $max_speed ) {
                $pos->vx( ( $vx / $speed ) * $max_speed );
                $pos->vy( ( $vy / $speed ) * $max_speed );
            }
        }
    }
}

```

Lastly we have the `Movement` system and the `Renderer` system. The `Movement`
system is the simplest of all the systems, it just updates the position of all
the entities with a `Position` component. And the `Renderer` system draws all
the entities we care about on the screen.

```perl
class Movement : isa(System) {

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            $entity->{'Position'}->update_location();
        }
    }
}

class Renderer : isa(System) {
    field $app : param;
    field $boid = Raylib::Text->new(
        text  => 'x',
        color => Raylib::Color::WHITE,
        size  => 10,
    );
    field $fps = Raylib::Text::FPS->new();

    method update ($ecs) {
        my $boid_count = Raylib::Text->new(
            text  => sprintf( "boids: %s", scalar $ecs->entity_count ),
            color => Raylib::Color::WHITE,
            size  => 10,
        );
        $app->draw(
            sub {
                $app->clear();
                $fps->draw();
                $boid_count->draw( 0, 20 );
                for my $entity ( $ecs->entities_with('Position') ) {
                    $boid->draw( $entity->{'Position'}->location() );
                }
            }
        );
    }
}

```

I really only included the `Render` system to show how you might use the ECS to
handle all of the parts of a game loop. In a real game you'd have a lot more
systems, and they'd be a lot more complex. But this is a good start.

We just need to wire everything up in the main app now.

```perl
my $app = Raylib::App->window( 800, 600, 'Boids' );
$app->fps(60);

my $ecs = ECS->new(
    systems => [
        Alignment->new(),
        Avoidance->new(),
        Cohesion->new(),
        ScreenEdge->new(
            width  => $app->width,
            height => $app->height,
        ),
        SpeedLimits->new(),
        Movement->new(),
        Renderer->new( app => $app ),
    ]
);

sub add_boid {
    my $entity = $ecs->add_entity();
    $ecs->add_component(
        $entity,
        Position->new(
            x => int rand( $app->width ),
            y => int rand( $app->height ),
        )
    );
    $ecs->add_component( $entity, Vision->new() );
    $ecs->add_component( $entity, Proximity->new() );
}

add_boid() for 1 .. 88; # max boids for my system before it starts to slow down
while ( !$app->exiting ) {
    $ecs->update();
}
```

And that's it. We have a simple boids simulation using an ECS system. It's not
perfect, and the constants are all wrong so it needs some tweaking, but it
demonstrates the concept of an ECS system and how you might use it to build a
simulation.


If you would like to learn more about how to leverage an ECS system to write a
video game, please come to my class at
[TPRC](https://tprc2024.sched.com/event/1d68A) in Las Vegas.
Tickets are [still available](https://tprc2024.sched.com/tickets), and I'd love to see you there!

Below is the full code for the boids simulation, it depends on `Raylib::FFI`
which is not released yet.

```perl
#!/usr/bin/env perl
use 5.40.0;
use lib qw(lib);
use experimental 'class';

use Raylib::App;

class ECS {
    field $systems : param;

    field @entities   = ();
    field %components = ();

    method entity_count { return scalar grep defined, @entities }

    method add_entity() {
        push @entities, {};
        return $#entities;
    }

    method add_component ( $entity, $component ) {
        $entities[$entity]{ ref $component } = $component;
        push $components{ ref $component }->@*, $entity;
    }

    my sub intersection (@lists) {
        my %u = ();
        return grep { $u{$_}++ } map { $_->@* } @lists;
    }

    method entities_with (@components) {

        # if we have no components, return all entities
        if ( $components{"@components"} ) {
            return grep defined, @entities[ $components{"@components"}->@* ];
        }

        # otherwise make a new archtype
        $components{"@components"} =
          [ intersection( @components{@components} ) ];
        return grep defined, @entities[ $components{"@components"}->@* ];
    }

    method update () { $_->update($self) for $systems->@* }
}

class Position {
    field $x : param;
    field $y : param;
    field $vx = rand();
    field $vy = rand();

    method location { return ( $x,  $y ) }
    method velocity { return ( $vx, $vy ) }

    method vx ($dvx) { $vx = $dvx }
    method vy ($dvy) { $vy = $dvy }

    method update_location() {
        $x += $vx;
        $y += $vy;
    }

}

class Vision {
    field $range : param : reader = 30;

}

class Proximity {
    field $distance : param : reader = 5;
}

class System {
    method update ($ecs) { ... }
}

class Avoidance : isa(System) {
    field $avoidance_factor : param = 1.5;

    method update ($ecs) {
        my @others =
          map { entity => $_, location => [ $_->{'Position'}->location ], },
          $ecs->entities_with('Position');

        for my $entity ($ecs->entities_with( 'Position', 'Proximity' )) {
            my $cdx = 0;
            my $cdy = 0;
            my ( $pos, $prox ) = $entity->@{ 'Position', 'Proximity' };
            my ( $x, $y )      = $pos->location();
            my ( $vx, $vy )    = $pos->velocity();
            my $sq_distance = $prox->distance**2;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                unless ( $dx**2 + $dy**2 > $sq_distance ) {
                    $cdx += $dx;
                    $cdy += $dy;
                }
            }
            $pos->vx( $vx + $cdx / $avoidance_factor );
            $pos->vy( $vy + $cdy / $avoidance_factor );
        }
    }
}

class Alignment : isa(System) {
    field $matching_factor : param = 0.08;

    method update ($ecs) {
        my @others =
          map {
            entity     => $_,
              location => [ $_->{'Position'}->location ],
              velocity => [ $_->{'Position'}->velocity ],
          }, $ecs->entities_with('Position');

        for my $entity ( $ecs->entities_with( 'Position', 'Vision' ) ) {
            # create some variables to store the average position, velocities and the number of boids in the vicinity
            my ($xpos_avg,$ypos_avg, $xvel_avg, $yvel_avg, $neighboring_boids);

            my ( $pos, $vis ) = $entity->@{ 'Position', 'Vision' };
            my ( $x,   $y )   = $pos->location();
            my $range = $vis->range;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                if ( abs($dx) < $range && abs($dy) < $range ) {
                    my ( $vx, $vy ) = $other->{velocity}->@*;
                    $xvel_avg          += $vx;
                    $yvel_avg          += $vy;
                    $neighboring_boids += 1;
                }
            }

            if ($neighboring_boids) {
                $xvel_avg = $xvel_avg / $neighboring_boids;
                $yvel_avg = $yvel_avg / $neighboring_boids;

                my ( $vx, $vy ) = $pos->velocity();
                $pos->vx( $vx + ( $xvel_avg - $vx ) * $matching_factor );
                $pos->vy( $vy + ( $yvel_avg - $vy ) * $matching_factor );
            }
        }
    }
}

class Cohesion : isa(System) {
    field $centering_factor = 0.0002;

    method update ($ecs) {
        my @others =
          map { entity => $_, location => [ $_->{'Position'}->location ], },
          $ecs->entities_with('Position');

        for my $entity ( $ecs->entities_with( 'Position', 'Vision' ) ) {
            # create some variables to store the average velocities and the number of boids in the vicinity
            my ($xpos_avg, $ypos_avg, $neighboring_boids);

            my ( $pos, $vis ) = $entity->@{ 'Position', 'Vision' };
            my ( $x,   $y )   = $pos->location();
            my $range = $vis->range;

            for my $other ( grep $entity != $_->{entity}, @others ) {
                my ( $other_x, $other_y ) = $other->{location}->@*;
                my $dx = $x - $other_x;
                my $dy = $y - $other_y;

                if ( abs($dx) < $range && abs($dy) < $range ) {
                    $xpos_avg          += $other_x;
                    $ypos_avg          += $other_y;
                    $neighboring_boids += 1;
                }
            }
            if ($neighboring_boids) {
                $xpos_avg = $xpos_avg / $neighboring_boids;
                $ypos_avg = $ypos_avg / $neighboring_boids;
                my ( $vx, $vy ) = $pos->velocity();

                $pos->vx( $vx + ( $xpos_avg - $x ) * $centering_factor );
                $pos->vy( $vy + ( $ypos_avg - $y ) * $centering_factor );

            }

        }
    }
}

class ScreenEdge : isa(System) {
    field $width : param;
    field $height : param;

    field $turn_factor : param = 0.02;
    field $margin : param      = 90;

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            my $pos = $entity->{'Position'};
            my ( $x,  $y )  = $pos->location();
            my ( $vx, $vy ) = $pos->velocity();

            if ( $x < $margin )           { $pos->vx( $vx + $turn_factor ) }
            if ( $y < $margin )           { $pos->vy( $vy + $turn_factor ) }
            if ( $x > $width - $margin )  { $pos->vx( $vx - $turn_factor ) }
            if ( $y > $height - $margin ) { $pos->vy( $vy - $turn_factor ) }
        }
    }
}

class SpeedLimits : isa(System) {
    field $min_speed = 2.0;
    field $max_speed = 4.0;

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            my $pos = $entity->{'Position'};

            my ( $vx, $vy ) = $pos->velocity();
            my $speed = sqrt( $vx * $vx + $vy * $vy ) || $min_speed;
            if ( $speed < $min_speed ) {
                $pos->vx( ( $vx / $speed ) * $min_speed );
                $pos->vy( ( $vy / $speed ) * $min_speed );
            }

            if ( $speed > $max_speed ) {
                $pos->vx( ( $vx / $speed ) * $max_speed );
                $pos->vy( ( $vy / $speed ) * $max_speed );
            }
        }
    }
}

class Movement : isa(System) {

    method update ($ecs) {
        for my $entity ( $ecs->entities_with('Position') ) {
            $entity->{'Position'}->update_location();
        }
    }
}

class Renderer : isa(System) {
    field $app : param;
    field $boid = Raylib::Text->new(
        text  => 'x',
        color => Raylib::Color::WHITE,
        size  => 10,
    );
    field $fps = Raylib::Text::FPS->new();

    method update ($ecs) {
        my $boid_count = Raylib::Text->new(
            text  => sprintf( "boids: %s", scalar $ecs->entity_count ),
            color => Raylib::Color::WHITE,
            size  => 10,
        );
        $app->draw(
            sub {
                $app->clear();
                $fps->draw();
                $boid_count->draw( 0, 20 );
                for my $entity ( $ecs->entities_with('Position') ) {
                    $boid->draw( $entity->{'Position'}->location() );
                }
            }
        );
    }
}

# main app

my $app = Raylib::App->window( 800, 600, 'Boids' );
$app->fps(60);

my $ecs = ECS->new(
    systems => [
        Alignment->new(),
        Avoidance->new(),
        Cohesion->new(),
        ScreenEdge->new(
            width  => $app->width,
            height => $app->height,
        ),
        SpeedLimits->new(),
        Movement->new(),
        Renderer->new( app => $app ),
    ]
);

sub add_boid {
    my $entity = $ecs->add_entity();
    $ecs->add_component(
        $entity,
        Position->new(
            x => int rand( $app->width ),
            y => int rand( $app->height ),
        )
    );
    $ecs->add_component( $entity, Vision->new() );
    $ecs->add_component( $entity, Proximity->new() );
}

add_boid() for 1 .. 88; # max boids for my system before it starts to slow down
while ( !$app->exiting ) {
    $ecs->update();
}
```
