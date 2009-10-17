Title: Classes Vs. Roles  
Author: Chris Prather
Date: 2009-05-28 17:37:41

# Classes Vs. Roles
Ovid [recently asked][1] if there was ever a reason to use Inheritance over
Composition. Stevan [answered][6] that question rather eloquently but the whole
thing has me thinking about the relationship between Classes and Roles.

Object Orientation has no bearing on the real world. It's framework for
building metaphors that hopefully help programmers better model reality. You
define Classes to represent some set of objects that reflect your problem
domain, and then use those classes to manipulate the domain to get answers or
control systems or [mutate weasels][2]. Roles come in to help compose
re-usable bits of behavior into the system. In languages without Roles this
composition of behavior is often achieved through the use of Abstract Classes,
or Classes that don't represent an object, but represent some quality of an
object.

To answer Ovid's question requires a deeper exploration of where Roles and
Classes differ. At first blush Classes and Roles are very similar. They both
bundle up a set of data and behaviors (sometimes this set is empty, see below)
a namespace or type for reusability. The difference, and this is crucial, is
that Classes can be instantiated and Roles cannot. That is Classes reflect
something concrete in your model, something that has an existence and can be
manipulated. Roles don't, they're always ethereal and abstract, and only gain
realization when composed with a Class.

Sometimes a Role and a Class do nothing more than establish a Type or
namespace. An example is the `NoGetopt` [trait][4] (A trait in Moose is
roughly a Role applied to a metaclass) and [metaclass][3] which are both
empty. The `NoGetopt` metaclass is a subclass of the Attribute metaclass with
the `NoGetopt` trait applied. Attribute metaclasses are part of the Moose
domain model, they are concrete and instantiated, the only way to replace a
metaclass is with something equally concrete. The ability to subclass
Moose::Meta::Attribute gives us the flexibility to apply the `NoGetopt` trait
at runtime. Otherwise we would have to design all the combinations of Roles
and Classes up front, or run into situations where we need to either implement
everything as a Role composed against an empty class (why have classes then?),
or run into situations where we would have to replicate a class to define a
new composition of roles (`Moose::Meta::Attributes::WithNoGetopt`) because there
are different situations where different combinations are needed in an instance.

Note that the `NoGetopt` trait exists so that you can apply it to an
anonymous subclass of a metaclass. This means that if you already have a
subclass of Moose::Meta::Attribute, Moose will (behind the scenes) create a
anonymous subclass and apply the `NoGetopt` trait to it.

Finally we can see this sort of logic play out in the ad-hoc way people choose
to decide the class/role relationship. Jonathan Worthington [commented][5] :

    "Dog does Walk" sounds much natural than "Dog is Walker" and that "Dog is Animal" 
    is more natural than "Dog does Animal"

Walking is *generally* an abstract concept, however we often say we are gonna
take the dog "for a Walk". Which suggests that in some models a Walk may be a
concrete concept. Relying upon ad-hoc "does it sound right?" heuristics means
that your model stops reflecting the needs of the domain. 

If a thing represents a piece of your domain you need to manipulate, and is a
specialization of another piece of your domain you need to manipulate then in
general you want to use inheritance.

[1]: http://use.perl.org/~Ovid/journal/39039
[2]: http://chris.prather.org/mutant-weasels/
[3]: http://cpansearch.perl.org/dist/MooseX-Getopt/lib/MooseX/Getopt/Meta/Attribute/NoGetopt.pm
[4]: http://cpansearch.perl.org/dist/MooseX-Getopt/lib/MooseX/Getopt/Meta/Attribute/Trait/NoGetopt.pm
[5]: http://use.perl.org/comments.pl?sid=43068&cid=68807
[6]: http://use.perl.org/comments.pl?sid=43068&cid=68803
