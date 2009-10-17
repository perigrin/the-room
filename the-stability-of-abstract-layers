Title: The Stability of Abstract Layers  
Author: Chris Prather
Date: 2009-05-31 18:33:58

# The Stability of Abstract Layers
One of the reasons that [Moose][1] really works is that it's a layer of
abstractions. At the bottom you have Perl's native OO layer which is ... let's
be charitable and say "bare bones". On top of that Stevan wrote
[Class::MOP][2] which standardizes the lower layer, cleans up the edge cases,
implements some best practices, and gives you a massive level of introspection
over everything. On top of [Class::MOP][2] is [Moose][1] which brings in a new
layer of features including Roles, Delegation, Type Constraints and the other
goodies you've heard so much about. On top of [Moose][1] are things like
[MooseX::Declare][3] and the rest of the MooseX:: namespace that can extend
the features that Moose provides in new directions.

This stack works precisely because it *is* a stack. Nested layers of
abstraction means you have a consistent base and you can play TIMTOADY on top
of it as much as you like but you get the consistency of a stable API at the
center. Eventually the new layers on top become a base for layers atop them.
The current discussion about the Perl release system seems to me to really be
about this idea. chromatic [pointed out][7] that Rakudo has had more stable
releases than Perl5 in the time since the Perl6 announcement was made. This is
true, but Rakudo's idea of stable is smaller stack of abstractions than
Perl5's. Rakudo doesn't ship 114 external modules. It doesn't even ship 100%
of the spec'd Perl6 features. Claiming that Rakudo's concept of stable is
comparable to Perl5's is disingenuous. I'm sure chromatic was trying to
highlight this discussion on what stability is rather than trying to troll
with statements full of FUD, he'd be the first to defend Perl if someone else
were to make this statement.

The right answer is probably a middle ground. Lots of people agree that Perl
should ship a smaller Core set of modules, perhaps a smaller set of core
features would help too, something to enable more externalizing of hooks like
[Devel::Declare][6]. I'm already failing at my volunteer effort to drive the
first part, I doubt I'd be the right person for the second (but I'm willing to
try).

This same idea comes up as well with the comparison of stability between
[Module::Build][4] and [Module::Install][5]. Those on the [Module::Build][5]
side point out that having a single API means that everybody is on the same
page and that you can trust all installers are equal. The [Module::Install][6]
people tend to point out that not all distributions are equal and that having
everybody conform to the same API causes problems when you try upgrading that
API (not everything is installed equally everywhere).

It seems to me that the answer is to have a core API (Module::Build, EUMM it
doesn't matter), and a stack of abstractions on top of that where you can do
experimental extensions. [Module::Install][8] currently does this on top of
EUMM.

[1]: http://moose.perl.org
[2]: http://search.cpan.org/dist/Class-MOP
[3]: http://search.cpan.org/dist/MooseX-Declare
[4]: http://search.cpan.org/dist/Module-Build
[5]: http://search.cpan.org/dist/Module-Install
[6]: http://search.cpan.org/dist/Devel-Declare
[7]: http://use.perl.org/~chromatic/journal/39046
[8]: http://search.cpan.org/dist/Module-Install
