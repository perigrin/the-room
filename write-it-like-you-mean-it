Title: Write it Like You Mean It  
Author: Chris Prather
Date: 2008-10-15 14:12:19

# Write it Like You Mean It
One of the things I've discovered recently, and wished I'd known *years* ago is you need to write all your Perl applications like you were gonna be posting them to CPAN, even if you have no intention of *ever* doing so.

At work we are starting to migrate from a legacy system that was written in the grandest of late 90s CGI (the code is clear, easy to read and for this era of script well documented, but lacks any architectural cohesion and has been patched and re-patched over the years to handle new use cases). This system is absolutely core to our daily business, you might even argue that it *is* our daily business. 

One of my tasks was to add Validation on some of the input parameters. My boss wants to move to something testable so I created a Validation module that contained a bunch of validation functions for the various pieces of data, and when we could easily do so we wrapped CPAN validation modules (Regexp::Common we love you!). This however added a couple of new dependencies to our system. This is where writing it like a CPAN package comes in.

I added a Makefile.PL for the system. We have no intention of *ever* releasing this code, but  the Makefile.PL uses Module::Install and lists the dependencies. The magic of this is that for recent versions of `cpan` you can just type `cpan .` and it will automatically install all the dependencies for the current directory. Failing that you can always run `perl Makefile.PL && make && make test` and it'll install dependencies and run your test suite.

By treating code we were never going to release to CPAN as if we were, we win the support of all of the CPAN toolchain. A toolchain that is getting better every day.
