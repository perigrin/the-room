---
layout: post
Title: If You Give a Mouse A Christmas
Author: Chris Prather
Date: 2009-12-02 18:14
---

Jerrad Pierce wrote a [Perl Advent Calendar entry][1] that has sparked
some discussion in the Moose community. He has revised it two or three
times, and the final version is disappointing but not blatantly wrong.
As a Moose contributor who has deployed Mouse into production I wanted
to give some perspective on the issues that came up.

Jerrad suggests strongly in the tone of his article that Mouse is
everything you want in Moose but faster. There are some strong issues
with this implication, the first being that there is no way to really
know if that is true. Jerrad quotes the Mouse documentation by saying

>    Mouse is "Moose without the antlers" i.e; lacking the thorny
>    dependencies and added heft giving you a pain in the neck.

But the truth is that the antlers the Mouse documentation is talking
about isn't the dependencies (which I've [blogged about previously][2]),
it is the Meta Object Protocol (the MOP). To quote Jerrad again (from
one of his re-writes)

>   [Mouse] can be a nicer, gentler introduction to the world of Moose.
>   Like the cervine form, the rodent provides a simple means of
>   providing accessors which are more explicit than a generalized
>   AUTOLOAD mechanism, while still eliminating redundant code. Plenty
>   of other fancy OO features come along for the ride, but no
>   "metaprotocol stuff," which some would argue is the raison d'être of
>   Moose.

Mouse implements much of the *sugar* from Moose, but implements the
bare-minimum of an Object System to back up that sugar.

>   You may be willing to make such a trade-off, but what if you're not
>   writing the code, and instead run into some other module that foists
>   Moose upon you? That author may or may not need all of Moose, but
>   chances are good they don't.

Here is where my problem with the solution provided in this calendar
entry comes. Chances are that the developer who chose to use Moose has
no clue which pieces of the MOP they rely upon. The MOP in Moose is a
kind of an iceberg, everything in Moose is build around it. The thin
sugar layer that most people interact with is only the surface of what
is going on inside.

This is the power of Moose. You don't *need* to know what is going on
for 90% of the things you do, you just need to know that Moose works,
and [MooseX::Aliases][3], or [MooseX::Getopt][4], or
[MooseX::Storage][5] can hook into the proper places in the MOP and get
things done.

Jerrad mentions this almost as an aside though in his calendar entry.

>   Note that even if other code will compile correctly with Mouse, it's
>   possible the code could be doing some deep introspection and you may
>   end up with Chet rather than Comet. it is therefore recommended that
>   you run the code's test suite against Mouse whether you force it
>   through Package::Alias or substituting Any::Moose.

How confident are you that your test suite includes the proper coverage
for your application that you can detect subtle bugs in the MOP? I have
a few applications that have high 90%+ test coverage, and I'm not sure
*they* would deal well with having their object system replaced
underneath them.

The fact of the matter is that unless the upstream module author
designed their application with possible Mouse usage in mind, it simply
cannot be a safe guaranteed drop in replacement for Moose. This is why
the Mouse documentation say to use Moose instead.

This leads to my last, and sadly personal, issue with this calendar post. I'm
an active member in the Moose community. I have been for about three years now.
I am one of the people who will have to support the kinds of failure that this
well intentioned but reckless advice will cause.

If Jerrad had simply talked to us when writing this calendar post, we wouldn't
have reacted so strongly. We could have pointed out the issues in the advice he
was giving, and helped him to write a better article that focused on the
important part of advocating Mouse as a reasonable replacement for Moose in
some circumstances, and the interesting hack he performed with Package::Alias,
or Alias.pm, or even the `import` hack someone in the Moose community
suggested.

[1]: http://www.perladvent.org/2009/1/
[2]: http://chris.prather.org/moose-dependencies--a-lurid-tale.html
[3]: http://search.cpan.org/dist/MooseX-Aliases
[4]: http://search.cpan.org/dist/MooseX-Getopt
[5]: http://search.cpan.org/dist/MooseX-Storage
