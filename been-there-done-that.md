Title: Been There, Done That
Author: Chris Prather
Date: 2010-03-14

# Been There, Done That

An email recently came up on the EPO-Marketing list that I made a long and rambling reply to. The ideas I expressed there I think are important and should get expressed to a larger audience so I thought I'd put them here.

The original poster to the email mentioned the recent excursion to CeBit. The impression was that the current marketing efforts while enthusiastic lack an articulate and compelling story for Perl as a solution, and tended to focus on specific Projects, specifically the vanguard projects of Catalyst, Moose, DBIx::Class, and Padre.

I agree. I think the problem is that we as a community, or the limited Modern Perl community, or even the few people with ideas and loud mouths, haven't really had the discussions about "next steps" in articulating the Modern Perl Story. We have still been dealing with the internal process of getting the the community to embrace Modern Enlightened Perl, and the need to promote Perl in general. 

So now we need to tell the Modern Perl Story for people who aren't inside the echo chamber. 

Moose, Catalyst, DBIx::Class, and Padre show large popular projects written in a Modern Enlightened style. A style that perhaps needs some articulation. From my perspective it is that they leverage Perl's cultural strengths: flexibility, distributed development, and strong conventions.

Flexibility in that they embrace TIMTOWTDI allowing you to customize and override, they choose configuration over convention at the code level. All three of these packages reflect this in the sheer number of their extensions. Catalyst::Plugin, MooseX, and DBIx::Class as top level namespaces all have dozens and dozens of modules under them above and beyond the core projects.

For distributed development not only do these projects have an open commit policy, anybody who shows up with a patch generally can get a commit bit, they also utilize the infrastructure CPAN brings to the table. They prefer to utilize CPAN dependencies over re-writing code when it makes sense, taking advantage of the 20,000+ packages in this 15 year old globally distributed environment. They also have very large test suites, and responsive core maintainers who watch the automatic testing that the CPAN Testers provide across at last count 58 different platforms.

Finally with strong conventions let me focus on the project I know the best of these three, Moose. Moose has worked hard over the last four years to build out more support for authors of patches. We have a documented support policy, a committers guide, and several author-specific tests that include code style, documentation, and optionally testing *all* down stream dependencies. We do this because, although we document that we priorities correct behavior over everything including backwards compatibility, our social conventions are to try maintaing compatibility where possible. We have our own best practices and idioms, and have had several long discussions about where things should be enforced via warnings and exceptions, and where we should leave them at the discretion of the author. Obviously we are not perfect at this, but we make the attempt and that in my biased opinion counts for a lot.

These three factors are not unique to these projects, they are instead the ideals that the community who surrounds these projects
strive for. They are, in my opinion, the ideals that the Modern Enlightened Community strives for.

There has been no formal research into why developers who have tried Perl choose another language. I'm not a social scientist, I have no clue how to begin such a project. I would love to have some research here. We can only speculate based on the comments we have all seen on various comment boards and have heard from co-workers and friends. On the other hand the largest reason I've heard for people who stay with the Perl community vs other communities is culture. This is articulated best in Piers Cawley's recent blog post [Falling out of love with a language][1], where he talks about going from Perl to Ruby and back. I'll let you read his excellent essay on it.

One of the things I've noticed in these discussions about Perl marketing is that we have unconsciously embraced the idea that Perl is losing ground somehow. I have held this idea too, at times. Recently however I've come to suspect that this isn't (yet) the case. 

This came up recently when Rob Diana posted [Web 2.0 Programming Language Job Trends – February 2010][2] to his Regular Geek blog. It was mentioned on twitter and when I looked at it I wondered why Perl wasn't included in the Web 2.0 languages. The reason is that Rob included Perl in the earlier [Traditional Programming Language Job Trends – February 2010][3]. 

When you add Perl to the first chart, it really stands out in comparison to most of the "Web 2.0" languages. In fact it's comparable only to Javascript in the chart, as you can see below.

<div style="width:540px">
<a href="http://www.indeed.com/jobtrends?q=ruby%2C+rails%2C+python%2C+php%2C+javascript%2C+flex%2C+groovy%2C+perl" title="ruby, rails, python, php, javascript, flex, groovy, perl Job Trends">
<img width="540" height="300" src="http://www.indeed.com/trendgraph/jobgraph.png?q=ruby%2C+rails%2C+python%2C+php%2C+javascript%2C+flex%2C+groovy%2C+perl" border="0" alt="ruby, rails, python, php, javascript, flex, groovy, perl Job Trends graph">
</a>
<table width="100%" cellpadding="6" cellspacing="0" border="0" style="font-size:80%"><tr>
<td><a href="http://www.indeed.com/jobtrends?q=ruby%2C+rails%2C+python%2C+php%2C+javascript%2C+flex%2C+groovy%2C+perl">ruby, rails, python, php, javascript, flex, groovy, perl Job Trends</a></td>
<td align="right"><a href="http://www.indeed.com/q-ruby-jobs.html">ruby jobs</a> - <a href="http://www.indeed.com/q-rails-jobs.html">rails jobs</a> - <a href="http://www.indeed.com/q-python-jobs.html">python jobs</a> - <a href="http://www.indeed.com/q-php-jobs.html">php jobs</a> - <a href="http://www.indeed.com/q-javascript-jobs.html">javascript jobs</a> - <a href="http://www.indeed.com/q-flex-jobs.html">flex jobs</a> - <a href="http://www.indeed.com/q-groovy-jobs.html">groovy jobs</a> - <a href="http://www.indeed.com/q-perl-jobs.html">perl jobs</a></td>
</tr></table>
</div>

But when you add Javascript to the Traditional Languages chart[^1] you see it fits in nicely with those languages.

<div style="width:540px">
<a href="http://www.indeed.com/jobtrends?q=java%2C+C%2B%2B%2C+C%23%2C+visual+basic%2C+Perl%2C+Javascript" title="java, C++, C#, visual basic, Perl, Javascript Job Trends">
<img width="540" height="300" src="http://www.indeed.com/trendgraph/jobgraph.png?q=java%2C+C%2B%2B%2C+C%23%2C+visual+basic%2C+Perl%2C+Javascript" border="0" alt="java, C++, C#, visual basic, Perl, Javascript Job Trends graph">
</a>
<table width="100%" cellpadding="6" cellspacing="0" border="0" style="font-size:80%"><tr>
<td><a href="http://www.indeed.com/jobtrends?q=java%2C+C%2B%2B%2C+C%23%2C+visual+basic%2C+Perl%2C+Javascript">java, C++, C#, visual basic, Perl, Javascript Job Trends</a></td>
<td align="right"><a href="http://www.indeed.com/q-java-jobs.html">java jobs</a> - <a href="http://www.indeed.com/q-C++-jobs.html">C++ jobs</a> - <a href="http://www.indeed.com/jobs?q=C%23">C# jobs</a> - <a href="http://www.indeed.com/q-visual-basic-jobs.html">visual basic jobs</a> - <a href="http://www.indeed.com/q-Perl-jobs.html">Perl jobs</a> - <a href="http://www.indeed.com/q-Javascript-jobs.html">Javascript jobs</a></td>
</tr></table>
</div>

I'm not sure what the implications of this are, except that the Perl and Javascript seem to trend closer to C# and C++ than they do Ruby and Python in the Job Market. This probably has some bearing upon how these languages are *used* in the market place and who our actual customers and competition are. What is certainly true here is that we're not in anyway really losing ground to anyone, yet.

Ultimately the real power of Perl isn't that we can implement any given solution better than another language. The real power is the cultural artifacts we express in the vanguard Modern Perl projects. It's that we allow configuration over convention at the code level, and that we are developing tools for enforcing cultural consistency. The biggest winner is that the odds are any given solution is already implemented and sitting on CPAN ready to be used. A real comparison between Modern Perl and any other language is is "write these dozen lines in your language" vs "Search search.cpan.org for someone else's solution".

Modern Perl's internal slogan has been TIMTOWTDI BSCINABTE (There is More than One Way to Do It. But Sometimes Consistency is Not a Bad Thing Either). I think if I were to suggest a slogan that Perl should have in the larger market place it's BTDT-IPAOC (Been There, Done That - It's Probably Already on CPAN).

[^1]: I have cleaned up this chart a bit removing Objective C and Delphi which really didn't trend well with the other languages and don't appear to trend well with the Web 2.0 languages either.

[1]: http://www.bofh.org.uk/2010/03/10/falling-out-of-love-with-a-language
[2]: http://regulargeek.com/2010/03/01/web-2-0-programming-language-job-trends-february-2010/
[3]: http://regulargeek.com/2010/02/02/traditional-programming-language-job-trends-february-2010/