---
layout: post
Title: State != Behavior  
Author: Chris Prather
Date: 2009-06-03 15:37:39
---

# State != Behavior
Ovid has [discovered][1] another "bug" (and by "bug" I mean documented
feature ;) ). He has code that effectively does:

    class MyClass {
        has my_method => ( default => 1);
    }

and then is surprised when `MyClass->new->my_method` fails. In his case it was
hidden by an inheritance hierarchy. This bites many people new to Moose, and there
has been a suggestion to make `has` default to something. The problem with a
default for `has` is that there are valid uses for *not* providing an accessor.

    class Toy {
        has sound_chip => (       
            default  => sub { SoundChip->new() }
            handles  => [qw(beep)],
        );
    }


Then you can say `Toy->new->beep`. We never want people to even know that a
`Toy` uses a `SoundChip` object because it's entirely encapsulated. The only
external exposure right now is someone could in theory say
`Toy->new(sound_chip => $geigerCounter)`. We can entirely encapsulate this
with:

    class Toy {
        has sound_chip => (
            init_arg   => undef, # disable new( sound_chip => Object )
            default    => sub { SoundChip->new() }
            handles  => [qw(beep)],
        );
    }

A more extended example (and closer to stuff I've seen in
production) uses Roles.

    role TemperatureAdjustments { requires 'adjust_temperature' }

    class Furnace {
        has thermostat => (
            does     => 'TemperatureAdjustments',
            handles  => 'TemperatureAdjustments', # proxy the entire role
            default => sub { StandardThermostat->new() }
        )
    }

then later you say:

    my $furnace = Furnace->new();
    $furnace->adjust_temperature("+1F");

Once the widget is integrated into the Furnace as a thermostat there is no
valid use case to access it directly. Providing a default for `has` means that
this is no longer simple and elegant. But! you say, shouldn't we optimize for
the most common behavior? Yes we should, and you'd be right to change the
default for `has` if it worked the way Ovid assumed it does.

    class MyClass {
        has my_method => ( default => 1);
    }

This starts with entirely the wrong assumption at `my_method`. You're not
declaring a method here, you are declaring an *attribute*. Attributes are
state and state != behavior. Moose allows you to *also* define behavior via
`has`. `is`, `reader`, `writer`, `clearer`, `predicate`, `handles`, and
`lazy_build` all enable some kind of behavior in your class.

By making a default for `is` what you're doing is creating action at a
distance and invisibly coupling the definition of behavior with the definition
of state.

UPDATE: Fixed some typos, thanks to Shawn Moore, Matt Trout, and Stevan Little for proofreading and editing.

[1]: http://use.perl.org/~Ovid/journal/39070
