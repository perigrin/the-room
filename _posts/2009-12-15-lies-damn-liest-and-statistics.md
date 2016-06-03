---
layout: post
Title: Lies, Damn Lies, and Statistics
Author: Chris Prather
Date: 2009-12-15 18:40
---

# Lies, Damn Lies, and Statistics

UPDATE: 

Aldo Cortesi contacted me privately via email with a response to this post. With his permission I've updated this post with the substance of his response and my own rebuttal. The updated sections are marked with UPDATE/END blocks.

END

So after chromatic [linked][1] to Aldo Cortesi's post [The impact of
language choice on github projects][2] I was curious. After reading the
post, I was shocked and dismayed at some of the analysis Mr. Cortesi
attempts to derive from his data. I brought it up on IRC and decided at
first that `rjbs`'s advice is best, "just ignore it."

Obviously that didn't stand, because I saw Flavio Poletti and osfameron
[both][3] [comment][4] about Cortesi's post. Upon second reflection I
realized that perhaps there was some nice value to be taken out of the
data he presents.

At the time Cortesi extracted his data from github, Perl was the 6th
most popular language.[^1] Cortesi's first two illustrations look
particularly nice for Perl. First he pulled out the median number of
contributors (emphasis is his).

>   __Most projects have around 3 contributors, with Perl and Java
>   projects having about 5, and Javascript and Objective C around 2.__

This means in general that Perl projects have *more* developers than the
other languages measured except for C++. This is a good sign. Next he
pulled out the median number of commits (again emphasis his).

>   __Most projects have around 75 commits.__

Perl however in his data (again like C++) has a much higher media,
nearly 200 commits median. His explanation for this was (to me) very
unsatisfying.

>   The Perl and C++ data, however, seems significant - projects in
>   these languages on average have a much longer commit history. I
>   suspect that this is due to a decline in popularity in these
>   languages. Recall that I collected data only for projects that had
>   recent commits. If fewer new projects are created in C++ and Perl,
>   we would expect projects in these languages to be older, on average.

I think that the sheer fact that Perl (and C++) are both much older
languages is enough to explain the discrepancy. Perl has many projects
that have converted from one or more version control systems (Moose for
example has a revision history that is only a year younger than Git
itself, and is considered a *young* project in the Perl world).

The Perl community also tends to get behind a few *good* projects. This
isn't really obvious unless you're active in the Perl community, there
are many many things that aren't obvious unless you're in the Perl
community, it actually is a problem but one for a later blog post.

So to extrapolate further, the Perl community tries to engender people
rather than create a *new* project, to use one of the 21 thousand
existing projects to solve their problem. This explains why we have a
higher median number of contributors to our projects as well as why our
projects are on a whole *older*. 

UPDATE: 

Cortesi pointed out the following.

> The first point where you strongly disagree with me is my conjecture
> that the higher median commit count for Perl and C++ is due to a
> decline in popluarity in the languages, which means that my sample is
> skewed towards "older" projects. You counter that it might just be that
> Perl and C++ are older, and that this is reflected in the project data.
> This doesn't sound right to me - for one thing, Perl was first releaed
> in 1987, and Python in 1990. I just don't think that a 3 year
> difference over a 20+ year history can explain the size of the effect.
> One way to approach this might be to look at the rate of project
> creation over the last few years, which would be easy enough to do.

He's right, I'm simply parroting the stereotype that Perl is
older than Python (or even Ruby which came into existence in the mid
90s) when I rightly know better. The stereotype however does have some
merit. It's generally accepted that Perl had it's *second* wind during
the dot-com era and that was when Python had it's first wind.

I'm honestly not sure that the data can be conclusive argued one way
or the other without perhaps correlating the age of the data (time
since first commit) in there as well.

END

Next Cortesi measured number of files touched per commit (emphasis his).

>   __Most commits touch about 4 files, with C++ touching somewhat more,
>   and Perl, Python and Ruby somewhat less.__

This is most easily explained by the nature of the dynamic languages
that more is done in smaller modules, blah blah blah. Neither Cortesi
nor I really found anything interesting in this because it upholds the
standard stereotypes.

Next Cortesi talks about Contributors (emphasis his).

>   __The average contributor contributes about 5 commits to a project.
>   C, Objective C and Ruby developers contribute somewhat less, PHP,
>   C#, Java and Javascript developers somewhat more.__ I suspect the
>   results for C and Ruby are due to projects in these languages
>   receiving more one-off contributions.

Here Perl clocks in at *exactly* the median. If I were to guess I think
the meaning here is that on average Perl projects have on average a
larger core group of contributers with smaller one off commits, and that
these two facts offset each other. This story reflects the information
above (higher median contributer count, older more established
projects), as well as the next point Cortesi makes (emphasis his).

>    __For all languages, a small fraction of the committers do the vast
>    majority of the work.__ This won't be news to anyone in the Open
>    Source community. More interesting, though, is the fact that __C,
>    C++ and Perl projects are significantly more "top-heavy" than those
>    in other languages, with a smaller core of contributors doing more
>    of the work.__

This really just reinforces the previous point that Perl's projects are
older and more established, with a stronger commitment from it's core
contributors. This could be read as Perl being a dying language, just
like C and C++. This could also be read as these three languages not
being _fad languages_, where people swoop in, commit a whole bunch and
then drift off. Really there isn't enough information here to tease out
which is true.

Finally Cortesi tries to make a correlation between the number of
committers for projects and something. I think however there may be a
problem with his analysis. First he measures commits vs committer as a
measure of how well languages recruit and retain contributors. Perl here
shows up fairly well actually, third only to Python and Ruby. I strongly
suspect that the significant discrepancy in data between these languages
(there are more than twice as many Python projects included as Perl, and
Ruby is four times again as much as Python) may come into play, but I've
no evidence to back that up.

UPDATE:

Cortesi noted the following, which is a fair point I overlooked. 

> Here, I'd like to point out that the graph shows the total commits vs
> the total committers per project. As such, it doesn't tell us anything
> about the _retention_ of committers - a committer who makes a single
> commit and leaves the project is still counted. I've been thinking
> about how to measure committer retention - perhaps looking at the
> timespan bracketed by the first and last commits of a committer, as a
> percentage of the lifespan of the project.

END

The next set of graphs shows the number of commits per day over the
first 300 days of a project. Perl here shows a strong decline, the
strongest actually. Coming in second is C, then Ruby being the last
obviously show a decline (Python and C++ could arguably show a decline,
but they just as arguably are flat). The other languages are all flat,
or show a slight increase.

A suggestion came up in the comments to Cortesi's that Perl projects
tend to be "complete" and move into maintenance mode. Cortesi actively
rejects this idea. While I disagree with his analysis[^2] I think he's
correct in refuting this. I *suspect* that instead Perl projects quickly
move to a point where they are obviously inferior to another project and
are abandoned.

This story would play out with the earlier arguments regarding the
commit count, and the age of Perl projects. The story behind
`DBIx::Class` is a good example (even though it works counter to my
argument). Originally Matt Trout used and actively contributed to
`Class::DBI`, `DBIx::Class` was intended to research ideas for moving
`Class::DBI`'s codebase forward. It wasn't until fundamental design
differences between `Class::DBI` and `DBIx::Class` were discovered that
the fork of the community really occured, and even then for the longest
time `DBIx::Class` maintained a CDBI compatible API.

This story plays out more often in the small, and in many of those the
research project is found to be fundamentally flawed, or folded back
into the parent project, or abandoned due to lack of community interest.
The fact that this story plays out again and again in the Perl community
is in some ways very ironic considering the Perl motto of TIMTOWDI, but
this is the basis of the new motto TIMTOWDI BSCINABTE (pronounced
Timtoady Bicarbonate and meaning There Is More Than One Way To Do It,
But Sometimes Consistence Is Not A Bad Thing Either).

Finally, it's obvious in the analysis and comments that Cortesi dislikes
Perl. He is allowed to have his opinion, obviously to anyone who reads
more than one post on this blog I don't share it. I found it
particularly dismaying[^3] however he let his opinion of Perl ruin his
analysis, and lead to what he calls "the venomousness of some of the
Perl commentators".

I can only assume that the venomousness he is talking about was removed
or particularly bad in email, because the comments that stand now
*appear* relatively rational and inline with the tone set in the body. I
can't pretend that the Perl community isn't full of loudmouth assholes,
I'm one of them, however if people find themselves feeling venomous,
they should take it to their own blog. Better yet, take a deep breath
and start that new project that is really cool and will change the
world, and upload it to github so that the world beyond Perl can see it.

UPDATE:

This is pure conjecture on my part, any opinion expressed here is entirely my own and has not been approved or vetted by Mr. Cortesi. 

Aldo seems genuinely surprised at the strength of reaction he had, and in speaking with him his comments weren't intended to be as inflammatory as they came off. I've expressed that the Perl of the past isn't the Modern Enlightened Perl we all know and love (YOU DO LOVE IT RIGHT?!?), and tried to show how things really are in the Perl universe I see rather than the one he's obviously experienced in the past.

END

[^1]: Today it is the second most popular language on github. The
increase in "popularity" is entirely based upon the [`gitpan`][6]
project which uploaded the entire [Backpan][5] repository as git repos.
However the 21,766 repositories in `gitpan` are not followed by at least
3 people each, so thankfully wouldn't affect the outcome of the
experiment.

[^2]: The conclusion that Cortesi makes is that Perl is that Perl is
somehow much more difficult a language to contribute to after 300 days,
but this is contrary to the *other* evidence he presents that perl
projects are in general older with a larger median contributor size, and
a larger core of contributors over time.

[^3]: I for one found the "offhand speculation" didn't even follow the
data presented accurately enough to justify its existence without a
qualifier (simply stating "my opinion is that ... "). A stronger
distinction between the analysis of the data, and the sterotypical
opinions he thought it backed up might have helped tone down the venom.

[1]: http://identi.ca/notice/16739116
[2]: http://corte.si/posts/code/devsurvey/index.html
[3]: http://www.polettix.it/perlettix/id_perl-5-parsing
[4]: http://greenokapi.net/blog/2009/12/15/github-language-statistics/
[5]: http://backpan.perl.org/
[6]: http://github.com/gitpan/