---
layout: post
Title: Sketchy Code
Author: Chris Prather
Date: 2009-07-30 16:31:00
---

# Sketchy Code
Quite often when I'm developing with a new framework or toolset I find it easy
to write a small test script to make sure I understand the feature I'm working
with. They're sort of like sketches artists do while working on a piece or
noodling an idea, or improve performances by actors trying to flesh out a
character.

Some code I posted earlier today to the [POE Cookbook][1] started out as a
sketch.
```perl
    #!/usr/bin/env perl
    use 5.10.0;

    {

        package Counter;
        use MooseX::POE;

        has count => (
            isa     => 'Int',
            is      => 'rw',
            default => 1,
        );

        has id => ( is => 'ro' );

        sub START {
            my ( $self, $kernel, $session ) = @_[ OBJECT, KERNEL, SESSION ];
            say 'Starting '.$self->id;
            $self->yield('dec');
        }

        event inc => sub {
            my ($self) = $_[OBJECT];
            say 'Count '.$self->id . ':' . $self->count;
            $self->count( $self->count + 1 );
            return if 3 < $self->count;
            $self->yield('inc');
        };

        sub on_dec {
            my ($self) = $_[OBJECT];
            say 'Count '.$self->id . ':' . $self->count;
            $self->count( $self->count - 1 );
            $self->yield('inc');
        }

        sub STOP {
            say 'Stopping '.$_[0]->id;
        }

        no MooseX::POE;
    }

    my @objs = map { Counter->new( id => $_ ) } ( 1 .. 10 );
    POE::Kernel->run();
```
I wrote this code initially to sketch out how [`MooseX::POE`][2] would work.
When I was writing it I had never used `MooseX::POE`, in fact nobody had since
I was still writing it. This code was a sketch to make sure that I understood
the interface I was developing and to make sure that it worked the way I
wanted.

Sometimes my sketches end up taking on a life of their own. The IRC bot Bender
on `irc.perl.org` started out as a sketch to learn `POE::Component::IRC` for a
project that has long since been abandoned. His is probably the oldest code
base that I developed that I still maintain, which is just to show you
sometimes the one you write to throw away never gets thrown away.

The final benefit I want to mention about code sketches is that they become
tests. The code above is part of the `MooseX::POE` [test suite][3] now, and
even if code doesn't become an official part of the test suite of whatever I'm
working on ... I can use it as a simple example of what I'm trying to achieve
so that I can ask others for help.

[1]: http://poe.perl.org/?POE_Cookbook/MooseX::POE
[2]: http://search.cpan.org/dist/MooseX-POE
[3]: http://cpansearch.perl.org/src/PERIGRIN/MooseX-POE-0.205/t/01_basic.t

