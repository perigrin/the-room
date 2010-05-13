Title: Fixing a Hole 
Author: Chris Prather
Date: 2010-05-12

# Fixing a Hole

Recently `xenoterracide` mentioned an issue he had with [Dist::Zilla][1]
which he solved in a [clever way][2]. I had been thinking about this
problem for a while myself. `RJBS` usually generates solid code, and
while we may not agree 100% on everything I respect him enough to give
serious thought to his solutions to problems. So it was surprising to me
that I had been looking at `Dist::Zilla` for a while and found something
about it unnerving that I couldn't put my finger upon. Then shortly
before `xenoterracide`'s blog post I came to the same realization he
had. 

`Dist::Zilla` is designed to help the author develop faster, but it
(inadvertently I'm sure) disenfranchises people who might contribute a
patch by raising the bar for contribution. There are extra hoops you
have to jump through to contribute. More modules from CPAN you need
before you can work on the modules from CPAN you need.

We have this problem with `Moose`, even though it uses
`Module::Install`. To hack on Moose you need roughly ten extra modules
installed that aren't required to run Moose. That is assuming you want
to properly test the results. I ran into this tonight when I was setting
up a smoker for Moose. My smoker couldn't just checkout the git repo and
start smoking it needed at the very least `Module::Install` and
`Module::Install::AuthorRequires`.

The solution I found was to simply embrace the problem.
`Task::SDK::Moose` is on it's way to CPAN. This is a module that will
install all of the modules you need to hack on `Moose` or `Class::MOP`
straight from the repository. To paraphrase [Homer Simpson][3].

>    Here's to CPAN: the cause of and solution to all of Perl's problems.

I'd like to thank `doy` for vetting the modules I included and making
sure `Class::MOP` was covered.

[1]: http://xenoterracide.blogspot.com/2010/04/my-new-lovehate-relationship-with.html
[2]: http://xenoterracide.blogspot.com/2010/04/distzilla-vs-xenoterracide.html
[3]: http://thinkexist.com/quotation/here-s_to_alcohol-the_source_of-and_answer_to-all/338899.html