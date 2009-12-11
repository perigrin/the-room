Title: Moose Dependencies: A Lurid Tale  
Author: Chris Prather
Date: 2009-05-01 00:00:00

# Moose Dependencies: A Lurid Tale
Right so the rumor is that `Moose` has a lot of dependencies. I mean according
to [cpantesters][1] `Moose` has 24 different dependencies! (Note: that same
site says that almost all of those dependencies have a nearly 100% test
record, but we're talking Quantity not Quality!). Let's examine this a bit
more:

[1]: http://deps.cpantesters.org/?module=Moose&perl=5.8&os=any+OS
 
Here are the actual direct dependencies from `Moose` and `Class::MOP`'s
`Makefile.PL`s

# Perl Core

* [`perl: 5.8.1` ](http://search.cpan.org/~jhi/perl-5.8.1/)

First Perl ... you must be at least version 5.8.1 to play, this means your
perl has to have been released sometime after September 2003

* [`Carp: 0`](http://search.cpan.org/~jhi/perl-5.8.1/lib/Carp.pm) 
    
`Carp` ships with 5.00 and above, so you've already got it
    
* [`Scalar::Util: 1.19`](http://search.cpan.org/~gbarr/Scalar-List-Utils-1.19/lib/Scalar/Util.pm)

`Scalar::Util` 1.19 ships with 5.8.9. `Moose` has a dependency on 1.19+ because of known issues in earlier versions. We re-export `blessed` as well as use `looks_like_number`, `refaddr`, `weaken`, and `set_prototype`
internally. 

# Moose Only

* [`List::MoreUtils`: 0.12 ](http://search.cpan.org/~vparseval/List-MoreUtils-0.12/lib/List/MoreUtils.pm)
    
We use `zip`, `first_index`, `any`, `all`, and `uniq`, these could be
re-implemented but they're in `List::MoreUtils` for a reason. Additionally the
versions here are written in C so there's very little performance hit.

* [`Sub::Exporter`: 0.972](http://search.cpan.org/~rjbs/Sub-Exporter-0.982/lib/Sub/Exporter.pm)

"Sub::Exporter - a sophisticated exporter for custom-built routines". Most of
Moose's advanced exporting features are based upon Sub::Exporter.

`Sub::Exporter` depends on `Params::Util`, `Sub::Install`, and `Data::OptList`.

* [`Data::OptList`: 0](http://search.cpan.org/~rjbs/Data-OptList-0.104/lib/Data/OptList.pm)

"Data::OptList - parse and validate simple name/value option pairs". We use
`mkopt` internally in Moose::Util. Since Data::OptList is a dependency of a
dependency already (`Sub::Exporter`) we can use it "for free".
    
# Class::MOP Only

* [`MRO::Compat`: 0.05](http://search.cpan.org/~flora/MRO-Compat-0.10/lib/MRO/Compat.pm)

"MRO::Compat - mro::* interface compatibility for Perls < 5.9.5". Perl 5.10.0
introduced a new `mro` pragma that made dealing with the inheritance tree much
easier. This module provides backwards compatibility for that pragma.

Depends on `Class::C3` which in turn depends on `Algorithm::C3` but neither of
which are used in Moose directly. Note that these two are baked into 5.10 and
thus this entire set of dependencies is a null op under Perls released since
2007.

# Non-Core -- Common between Moose & Class::MOP

* [`Devel::GlobalDestruction`: 0](http://search.cpan.org/~nuffin/Devel-GlobalDestruction-0.02/lib/Devel/GlobalDestruction.pm)

"Devel::GlobalDestruction - Expose PL_dirty, the flag which marks global
destruction." We expose this as `Class::MOP::in_global_destruction`, though
that is undocumented.

Depends on `Scope::Guard` and `Sub::Exporter`. 

"Scope::Guard - lexically scoped resource management". This creates a guard
object, which is effectively an object that triggers a cleanup function when
it is garbage collected.

* [`Sub::Name`: 0.04](http://search.cpan.org/~xmath/Sub-Name-0.04/lib/Sub/Name.pm)

"Sub::Name - (re)name a sub". We expose this as `Class::MOP::subname`, which
is used extensively internally in both `Class::MOP` and `Moose`.

* [`Task::Weaken`: 0](http://search.cpan.org/~adamk/Task-Weaken-1.02/lib/Task/Weaken.pm)

"Task::Weaken - Ensure that a platform has weaken support". `Scalar::Util`
sometimes comes in a pure perl variant that lacks the `weaken` function that
Moose uses. This ensures that feature is present even on broken installations.

# Removed Dependencies

* `Sub::Identify`

This has been folded into `Class::MOP` directly. It was used to maintain Pure
Perl compatibility but that effort was abandoned when maintenance became a
nightmare and we discovered we were never gonna have 100% compatibility
between Pure Perl and XS.

* `Test::Output`: 0.09

This has become optional.

# Required for Tests

`Moose` and `Class::MOP` have over 5519 core tests. We depend heavily on
`Test::More` and `Test::Exception` and several *optional* modules that if you
have them installed will run extra tests.

* [`Test::More`: 0.77](http://search.cpan.org/~mschwern/Test-Simple-0.86/lib/Test/More.pm)

"Test::More - yet another framework for writing test scripts." `Test::More` 0.80 ships with Perl 5.8.9.

* [`Test::Exception`: 0.21](http://search.cpan.org/~adie/Test-Exception-0.27/lib/Test/Exception.pm)

"Test::Exception - Test exception based code". Provides `lives_ok`,
`throws_ok`, `dies_ok` and `lives_and` that are used extensively in the
`Class::MOP` and `Moose` test suites.

* [`File::Spec`](http://search.cpan.org/~smueller/PathTools-3.29/lib/File/Spec.pm)

"File::Spec - portably perform operations on file names." This shipped with
5.004. You have it installed.

