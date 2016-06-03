---
layout: post
Title: Accessors, what's in it for Me
Author: Chris Prather
Date: 2009-11-13 15:14
---

# Accessors, what's in it for *Me*

So there has been some discussion recently on various places I almost never go to on the internet. That has bled over into #moose-dev, the developer channel for Moose.

The argument came up apparently that there's no point to an accessor that is just a glorified wrapper around hash key assignment. The basic assumption is that Moose's accessors are basically:

    sub foo { shift->{foo} = shift; }
    
Which isn't true. But let's assume for a moment it is. What does this buy us over simply saying `$obj->{foo} = $value`. At first glance not much. Now the Moose crowd's answer was "but we can add a ton of extra stuff in there for validation, and coercion", and that's true but that is missing the fundamental point. Encapsulation. People using your Object should never have to know if they're accessing state or behavior.

What if I decided to re-factor my object so that `foo` is now a delegate to a `FooThing` object.

    sub foo { shift->foo_thing->{foo}} = shift; }
    
If I'm using direct hash access I have to go and update *every single line of code* that uses `$obj->{foo}`. That's broken. The point to encapsulation is that I can change state to behavior, or behavior to state in *one* place and not have to fix any of my client code because it's safely abstracted behind an API.

Perl5 has several ways to fix this. You can do what Moose does, and make it dead simple to just make accessors for *everything* and build a up a culture that smacks you on the wrist if you violate encapsulation. Alternatively you can go out of your way to hide the fact that you're changing state to behavior, or behavior to state by using `tie` or lvalues. 

One way is simple, and well tested. The other way is full of pitfalls and increases the amount of "Magic" in the system by using obscure creatures from the Perl5 bestiary.

