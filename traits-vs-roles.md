Title: Traits vs Roles
Author: Chris Prather
Date: 2010-08-11

# Traits vs Roles

In natural languages lot of objects have multiple names. For example I
am known on CPAN and IRC as `perigrin`, I am know to my friends and
family as Chris, to the TSA as David, and to my children as Daddy. Each
of these different names illuminate a different context or role that the
object, me, is thought of or used in.

Recently there was a debate or argument about the concept of Traits vs
Roles in `#moose`. This has been a issue for a [good long
while][so-traits]. Ultimately the answer is that Traits are another name
for Roles. Just as I am both `perigrin` and Chris, Traits and Roles both
refer to the same concept but with different context or conceptual
baggage allowed.

But this isn't really a satisfying answer for a programmer. Two names
for a single concept isn't very efficient, especially if the
differentiation isn't very concrete. For example Chris vs `perigrin` is
(in theory) cleanly delineated by Offline vs Online. I think a little
history will help explain the reason that these two names exist and are
adopted by the Moose community.

First the concept of Roles is generally based upon a paper 
[__Traits —Composable Units of Behavior__][scg]. For this paper they created an
implementation of what we call Roles for Smalltalk 80, and named them
Traits. So outside of the Perl world, Roles are Traits. This is the
first point of confusion.

Next, like many good things in Perl these days, Traits and Roles started
in Perl with Perl6. Roles in Perl6 are effectively identical to Perl5,
however Perl6 has a specialized concept under the name 'traits'.
According to [Synopsis 06][syn06]:

>    Compile-time properties are called "traits".The is NAME (DATA)
>    syntax defines traits on containers and subroutines, as part of
>    their declaration:

>     constant $pi is Approximated = 3;   # variable $pi has Approximated trait
>     
>     my $key is Persistent(:file<.key>);
>     
>     sub fib is cached {...}
     
If you follow the idea that in Perl6 everything is an object, these
traits at compile time modify a given instance. In the examples above
the `$pi` instance of the constant class is modified to have a
Approximated trait that is set to 3, the `$key` object is made
persistent to a `.key` file, and the subroutine instance `fib` is given
the `cached` trait for presumably memoization.

Synopsis 6 goes on to state:

>    Properties are predeclared as roles and implemented as mixins--see
>    S12.

So at some level in Perl6 Traits are Roles that are applied to
instances. This is the second point of confusion.

Finally in the evolution of Moose we, that is the people in #moose at
the time, originally tended to only refer to Traits to Roles that
applied to metaclass instances. Traits in this case were things like
MooseX::Storage's 'DoNotSerialize'. This trait you can apply to a
attribute and that attribute is skipped when MooseX::Storage does it's
serialization routine. These traits are used all over, MooseX::POE is
implemented as just such a trait. Mostly they do their work hidden in
the Meta Object Protocol.

The logic behind calling these things Traits was two fold, one if you
consider the concept of a trait from perl6 it is conceptually similar to
a Role applied to an instance. and since Metaobjects in Moose are
themselves just instances that define things like Classes, Attributes,
Methods, and Roles the idea of a Role that applies instances being
called a Trait is small step. At the time we didn't have a lot of places
where people *generally* applied Roles to non-metaobjects instances.
This was before MooseX::Traits was written, before
MooseX::Object::Pluggable (the basis for Devel::REPLs plugin system),
before a lot of things we have now.

So where do we stand now? We have this concept, "a composable unit of
behavior", that has two names: Roles and Traits. Is this confusing? Yes,
somewhat. It's messy and confusing to new people.[^1] Is it useful? yes,
somewhat. Just like having multiple names for myself is useful to help
differentiate context[^2], having different names for subtly different
contexts is an "Advanced Feature". Like so much in Perl you can choose
not to use it and refer to everything as a Role, or a Trait, or a
"composable unit of behavior".

[^1]: Confusing the context of when to use which name is always
confusing, as is using the wrong name in the wrong context. Try being
woken up at 3am with a loud obnoxious brit referring to you by your IRC
name! One of the more confusing moments in my life.

[^2]: I know that if someone on the phone is calling me `perigrin` they
are talking about Perl, or if they're calling me `David` they've never
actually spoken to me before and got my name from some official document
somewhere.

[so-traits]: http://stackoverflow.com/questions/1093506/how-do-roles-and-traits-differ-in-moose/
[scg]: http://scg.unibe.ch/research/traits
[syn06]: http://perlcabal.org/syn/S06.html#Properties_and_traits