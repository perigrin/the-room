Title: Everybody Wants Some … CPAN
Author: Chris Prather
Date: 2009-11-10 12:04

# Everybody Wants Some … CPAN

So the twitters were all a … well twitter … with the [People want CPAN :-)][1] thread. This had some amazing synchronicity with the Perl talks we gave at [Big Lamp Camp][2] in Nashville this weekend. The main thrust of [Stevan][3] and Cory's talks were "It's the CPAN stupid."

However reading through the Python thread I've seen some places where people are ignorant of the reality of CPAN.

## CPAN is a set of hierarchical servers

The core CPAN ecosystem is a filesystem and three index files. I have a script I borrowed from someone that lets me mirror the entire CPAN repository around using just `rsync`. The CPAN tools will work on this local filesystem as well as they do on any of the remote ones. There is no hierarchy there is simply an acyclic digraph.

## There is one unified distribution format.

This depends on how you look at it. There are at least three major solutions for building a CPAN package each having it's own affordances and constraints. For example all of my modules use `Module::Install` because I find the declarative syntax useful and like the fact that it bundles the installation requirements along with the package. However some people *loath* the fact that it bundles itself and prefer `Module::Build` which is a pure perl solution, while yet others rely upon `ExtUtils::MakeMaker` (which `Module::Install` uses internally).

The fact that users don't even notice that this difference exists however is awesome. This is the whole point of TIMTOWTDI BSCINABTE (There is More Than One Way To Do I, But Sometimes Consistency Is Not A Bad Thing Either). We have one consistent interface for three competing technologies.

## There is a single consistent interface

Actually there are at least two clients for CPAN, `CPAN.pm` and `CPANPLUS.pm`. Each comes with it's own command line script, `cpan` and `cpanp` respectively. And, just like the distribution formats, they have radically different guts with different constraints and affordances.

They both however work rather identically from the client perspective. 

    cpan -i Moose  # install Moose
    cpanp -i Moose # install Moose

## There is a BuildBot on the Server

CPAN Testers are brilliant. They like everything in Perl are a wholly distributed volunteer effort. The two cpan clients both have supported uploading a snapshot of the test results for every run of a test suite for almost as long as I can remember. One day someone decided to build a script that just downloaded every module uploaded to CPAN, run the installation up to the test suite and report it. Then someone else got in on it, and someone else … eventually it became a small contest to see who had the most reports covering the most platforms. Now they cover (as of last month) 13 Perls on 11 operating systems across 57 platforms. 

The quality of these tests is somewhat suspect too. There are no coverage requirements, and code coverage doesn't really mean anything. The Perl community has also gone through several testing related fads with regards to developer vs. end-user oriented tests. There are some debates still going on about the "opt-out" nature of installing CPAn packages because some large dependency chain installs can be obtuse for the end-user to debug if a test fails somewhere deep in the chain.

The fact that Testing is now part of the defacto CPAN ecosystem is awesome but the important part isn't the "BuildBot", it's the collection and aggregation and reporting of the statistics. 

## CPAN Deals with Versioning

This is pretty much false. CPAN relies upon Perl's ability to detect version numbers and Perl is incredibly simplistic. As long as a version can be numerically compared as greater than another version that's about all the promise CPAN can give you.

CPAN in fact can make matters worse because it assumes that the computationally greatest version number on CPAN is the "best" version for a given package. The CPAN ecosystem has no way to specify a dependency on only version N of a package.

## CPAN is Awesome, but CPAN sucks

CPAN is a loosely organized large pile of semi-tested crap. It follows [Sturgeon's Law][4] that 90% of everything is crud pretty closely. But with the current size of CPAN that means that there are approximately 1,917 packages that aren't crud. Finding these packages is a continual conundrum. 

The fact is however that if the problems are social (should testing be opt-in or opt-out? how do you find a good package? what do you do about versioning?) then the technical solution is doing it's job incredibly well. In a very significant way CPAN *is* the Perl language, without it you couldn't have a post-modern Object System, simple Tail Call Optimization, Parser Level syntax extensions, or any number of other low level adaptations to the language.

## UPDATE

So apparently people found this post interesting and I've had a few people metion problems with it. I've fixed the `cpan`/`cpanp` command line since they both will accept `-i` but only `cpan` will let me do `cpan ModuleName`.

Also I've been told to link to [The Zen of Comprehensive Archive Networks][5] which explains in far more detail how CPAN works, and was written by the masterful Jarkko Hietaniemi.

[1]: http://article.gmane.org/gmane.comp.python.distutils.devel/11359
[2]: http://enterpriselamp.org/camp/
[3]: http://twitpic.com/onfv1
[4]: http://en.wikipedia.org/wiki/Sturgeon%27s_Revelation
[5]: http://www.cpan.org/misc/ZCAN.html
