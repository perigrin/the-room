Title: I Did it the "Right" Way  
Author: Chris Prather
Date: 2009-09-07 16:33:15

# I Did it the "Right" Way
Okay the last thing that came up in Dave Cross's now infamous Moose or No
Moose post. This post comes last because I noticed the question last but it
also took the longest to write. The questions is about the idea that is Moose
the One True Way. To quote the original Poster:

<blockquote>
Define "do OO right". Any answer containing explicit reference to Moose will
be disqualified.
</blockquote>

So let's take up this challenge. What does it mean to "do OO right" with Perl5?

Turns out nobody has actually defined this. An Object is basically four things:

* Abstract / Encapsulated Data
* Method Dispatch System
* Class Inheritance
* Subtype Polymorphism 


## Abstract / Encapsulated Data

Abstraction or Encapsulation is the idea that there is a distinction between
what you present to the User, the people or things using your object, and what
your object needs to keep track of internally. There are things that your user
*must* know about to be able to properly use your object, but there are things
that your user should never need to know.

For example one of the things a user *must* know about is what "state" the
object is in, the user rightfully expects an object to be able to answer basic
questions like "What is the user's name?" "How many items are in the list?"
"Is the color of this widget blue?". A user *must* know that the data coming
back from such questions is valid to some degree. The object's public API
makes these promises. The objects private implementation is how these promises
are kept.

Let's look at an example.

The data structure that keeps the "state" of an object is called an "instance".
Perl's default OO creates an "instance" by making a reference to a data
structure "special" using the `bless` command. Looking at the current most
popular "object system" on CPAN, Class::Accessor, it assumes you're using a
blessed hash reference for your instance. Stealing an example from their POD
they in fact assume your Class will look something like:

    sub name {
        my $self = shift;
        if(@_) {
            $self->{name} = $_[0];
        }
        return $self->{name};
    }

    sub salary {
        my $self = shift;
        if(@_) {
            $self->{salary} = $_[0];
        }
        return $self->{salary};
    }
    
    # etc...

Is this the *right* way in some philosophical way? That's arguable. For
example there is nothing in here making sure that the data you are storing in
your instance is valid.

Is this the most popular method? Yes. Not only does Class::Accessor assume
that you're using this style of object instance, but the first ten hits on
Google for "Perl Object Orientation" take you to tutorials explaining how to
set up this style of instance. So is this the right way? If you want to
be interoperable with CPAN and do what other Object Oriented Perl developers are
expecting ... yes probably.

Moose by default uses a blessed hash for instance data. So there's no benefit
to Moose here, but no cost really either. Moose however, like Class::Accessor,
provides a way to create much of this boiler plate for you. The above
definitions in Moose would be:

    has name   => ( is => 'rw' );
    has salary => ( is => 'rw' );
    
Making the definition of this boiler plate code much much simpler and arguably
clearer. This is what Class::Accessor provides for you as well, as well as
many of the other object systems. So no real benefit to Moose here over the
other method generators. However Moose also provides a system of defining
validation though called Type Constraints. If we change our example to

    has name => ( 
        isa => 'Str',
        is  => 'rw',
    );
    
    has salary => (
        isa => 'Int',
        is  => 'rw',
    )
    
The generated code would look much more complex. For more here look at
[`Moose::Manual::Unsweetened`](http://search.cpan.org/dist/Moose/lib/Moose/Manual/Unsweetened.pod)
for a more complete example of what Moose provides here.

Is this the *right* way? Making sure your data is valid and encapsulated is
certainly more correct than *not*. If you value encapsulation and validation
of your data, Moose has it baked in. I'm not aware of another object system on
CPAN that has validation baked into it but I don't keep track of all the
packages on CPAN either[1]. This isn't even the surface of the tools that Moose
has available for organizing and defining instances and making sure that
everything is consistent.

Moose works very hard to make sure that the public parts of your Object are
guaranteed up front while being very lazy about the private parts of your
implementation. This means that Moose tries let you implement as much (or as
little) as you needs to keep the promises your object makes. There are many
more features of Moose attributes that help expand on this idea as well,
including `required`, or `lazy` attributes and `builder` methods to help
default proper defaults.


## Method Dispatch System

In Object Orientation each instance is associated with a collection of methods
in a namespace called a Class. In Perl this is done with the same `bless`
keyword that marks an reference as "special". Perl re-uses the `package`
namespaces it uses for Modules to gather the methods for instances.

There aren't many packages on CPAN that really muck with this. There is
[`autobox`](http://search.cpan.org/dist/autobox) that will allow you to call
methods on unblessed references, and there are some experimental packages that
will let you override the method dispatch system that looks up the method
associated with an instance.

Is this the *right* way to do dynamic method dispatch? This is the *only* way
as far as I know that Perl does dynamic method dispatching. So we don't really
have a choice.

Moose does everything the same way as vanilla Perl here. Moose also provides a
set of tools to "wrap" methods and provide context. These Method Modifiers as
they are called are used to extend or sometimes override the normal method
dispatching.

    package A;
    use 5.10.0; # enable 'say' and other 5.10-isims
    use Moose;
    
    sub method_a { say 'method a' }
    
    before method_a => sub { say 'before' };
    after method_a => sub { say 'after' }; 
    
    package main;
    use strict;
    
    my $object = A->new();
    $object->method_a;
    
this will output

    before
    method a
    after

I'm glossing over some details here in preference of brevity but the fact is
these Method Modifiers can be very powerful tools for adding extra effects to
a class without having to completely re-write the method.

## Inheritance / Delegation

Inheritance or Delegation mean that when Perl goes to look up a method in the
Class associated with an object, if that method is not found Perl will then
follow a process to check *other* namespaces to see if the method is found in
one of those. This is usually best shown by an example, using standard Perl:

    package A;
    use 5.10.0; # enable 'say' and other 5.10-isims
    use strict;
    use warnings;
    
    sub new { return bless {}, 'A' }
    sub method_a { say 'method A' }
    
    package B;
    use strict;
    use warnings;
    our @ISA = qw(A); # Inherit from A
    
    package main;
    use strict;
    use warnings;
    
    my $object = B->new();
    $object->method_a; # this will print "method A";


What happens is that at runtime Perl will check the B namespace to see if a
subroutine named 'method_a' exists. If it doesn't it then starts walking
through the @ISA array in the B namespace for more namespaces to check.

Is this the "right" way? It's the default way in Perl. Perl ships with modules
like `base` and `parent` that will take care of the @ISA logic for you.

Moose uses this same system, instead of `base` or `parent` Moose provides a
keyword `extends` to define the @ISA array.

Moose also adds a system called Roles to the mix. Roles are like partial
Classes and when Moose is done setting things up more or less everything in a
Role becomes part of the Class. There are rules for composing Roles into
classes that aren't important here, the important bit is that Moose provides a
new level of flexibility in designing Classes and inheritance that you don't
have in standard Perl.

## Subtype Polymorphism 

This means that for any given piece of code that operates on a type of
instance can operate on a different instance that is the same type or a
subtype. Basically that in the example above anything that operates on
instances of Class A should also be able to operate on instances of Class B
without knowing or caring that they're different types since B is a subtype of
A. In fancier circles this is known as 
[the Liskov substitution principle](http://en.wikipedia.org/wiki/Liskov_substitution_principle).

Perl provides an `isa` method to check this. So lets say we are writing an
accessor to store a Customer object. Using Vanilla Perl you might write this
out like:

    sub customer { 
        my $self = shift;
        if (@_) {
            die "Not a customer" unless $_[0]->isa('Customer');
            $self->{customer} = $_[0]
        }
        return $self->{customer};
    }


Is this the "right" way? No. Is it a common way? Yes, very. 

Why is this not the right way? Well first we are assuming that people will
only ever pass objects into customer(), and that that object has an `isa`
method. Since Perl guarantees that every object has an `isa` method the second
part is okay, but the first part breaks our code.

    $object->customer({name => 'Jon'});
    
Will die quickly complaining that it can't call `isa` on an unblessed
reference. To fix this we can say in Vanilla Perl

    sub customer {
        my $self = shift;
        if (@_) {
            die "Not an Object (we think)" unless ref $_[0] && ref $_[0] =~ /HASH|ARRAY|SCALAR|GLOB|CODE/;
            die "Not a customer" unless $_[0]->isa('Customer');
            $self->{customer} = $_[0]
        }
        return $self->{customer};
    }

This checks to see if the value passed in is a reference and that reference
isn't one of the standard references. Now mind you this doesn't work if
someone happens to bless their object into a package named after one of the
valid reference types (this does happen). Luckily Perl ships with a module now
called `Scalar::Util` that has a `blessed` function, which works exactly like
`ref` but if the value is not blessed it returns undef meaning we can rewrite
this to

    use Scalar::Util qw(blessed);
    
    sub customer {
        my $self = shift;
        if (@_) {
            die "Not a customer" unless blessed $_[0] && $_[0]->isa('Customer');
            $self->{customer} = $_[0]
        }
        return $self->{customer};       
    }
    
This is close to the "right" way to set up a validating accessor in Perl. I'm
probably forgetting something, and this only works on the simplest case. In
Moose this would be written

    has customer => ( 
        isa => 'Customer',
        is  => 'rw',
    );

In addition because Moose has a concept of Roles, instances can have more than
one type. This is similar in concept to how Java's Interfaces work (and the
reason that analogy is often used when explaining Roles)

    has customer => (
        does => 'Role::Buyer',
        is   => 'rw',
    );

You can even intelligently combine the two options.

    has customer => (
        isa  => 'Customer',
        does => 'Role::BigSpender',
        is   => 'rw',
    );

Moose will sort out the details and create the right validation code.

This is probably the biggest reason why we claim that using Moose is doing
Perl OO the *right* way. Because it codifies all of the logic that 62
different developers have discovered over the course of their careers as being
the "right" way to set up accessors and do validation and makes sure that it
is the easiest way to do things by default.

## Everything Else

Now I think I've made a reasonable attempt at defining the "right" way to do
Perl OO, and explained how that is how Moose works by default without using
Moose in the definition only in the explanation. At this point Moose could
stop and not have severe start-up penalties and be called Mouse.

But Moose is built around something called Class::MOP. Dealing with the low
levels of manipulating raw symbol tables in Perl has always been considered
advanced knowledge, but Class::MOP is an Object Wrapper around that. A wrapper
designed to build Object Oriented code.

This means that Moose itself is built out of objects and those objects are
introspect-able and replaceable. This means that in theory (and generally in
practice too), you can replace any piece of Moose with code that operates
differently. This is how most of the Moose extensions (those modules starting
with MooseX) work.

So not only are the defaults as close to what the "right" way to do OO as
those of us who have committed to Moose can come up with, if we're wrong you
can in fact replace *just* the bits you disagree with us about.

[1] It turns out that [Class::Meta](http://search.cpan.org/dist/Class-Meta), [Class::Maker](http://search.cpan.org/dist/Class-Maker), and [Class::Tanagram](http://search.cpan.org/dist/Class-Tanagram) all have some form of validation baked in. In addition I wasn't thinking about [Mouse](http://search.cpan.org/dist/Mouse) or [Coat](http://search.cpan.org/dist/Coat) which are both based on Moose and have ported a variation of Moose's TypeConstraints. I really wasn't thinking very clearly when I made that comment.


