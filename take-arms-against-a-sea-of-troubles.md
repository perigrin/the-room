Title: Take arms against a sea of troubles  
Author: Chris Prather
Date: 2009-09-01 22:58:22

# Take arms against a sea of troubles
So [Dave Cross][1] has started injecting Moose Koolaid into his CPAN modules
and got smacked by [Adam Kennedy][2] for injecting Moose into something with a
ton of downstream dependencies. Or at least it *used* to have a ton of
downstream deps, the module in question, [Test::Warn][3] (as Adam rightly
points out) has moved on. 

Adam's Moose complaints are two fold. It "adds a huge amount of
additional dependencies", and "the slower process startup time". These are the
two standard complaints about Moose, along with "Ahh My RAM!!!".

I've blogged about Moose's dep graph before, and I'll simply refer you to some
charts I generated for Moose using CPANDB [(runtime][4], [5.8.1][5], [5.10.0][6]).

The point here isn't the "Ahh! Moose!" sentiment really. I thinks Stevan
[answered][7] that quite well. The point is who is generally complaining about
the slings and arrows of outrageous dependencies. 

In the bug in particular we have two commentors, Adam who opened the bug, and
has complained before about Moose's dependencies and overhead. Adam however
gets a pass because Adam has made at least some minimal effort to help [in the
past][8]. I don't believe as of this writing anybody else on that ticket
[has][9].

Moose has a very open commit policy, if you want a patch there are about 10
people who can give you access to the Moose git repository (I am one of them).
There are currently 62 people with access to the main git repository. Moose
also is mirrored on [Github][10]. We have a documented [contribution][11]
policy that goes into detail about how to submit large complex refactors for
review. There are over 200 people on `#moose` on `irc.perl.org`, and 45 people in
`#moose-dev` all of whom are more than willing to help fix the known issues with
Moose.

So if you have a problem with Moose, what can I do to help you fix it?

[1]: http://perlhacks.com/2009/09/moose-or-no-moose.php
[2]: http://rt.cpan.org/Public/Bug/Display.html?id=49270
[3]: http://search.cpan.org/dist/Test-Warn

[4]: http://chris.prather.org/graphs/Moose-runtime-5.010.svg
[5]: http://chris.prather.org/graphs/Moose-5.008001.svg
[6]: http://chris.prather.org/graphs/Moose-5.010.svg

[7]: http://stevan-little.blogspot.com/2009/09/re-moose-or-no-moose.html
[8]: http://git.shadowcat.co.uk/gitweb/gitweb.cgi?p=gitmo/Moose.git;a=commit;h=3c2bc5e2dc448e36704a71f25d66503cef8831fb
[9]: http://git.shadowcat.co.uk/gitweb/gitweb.cgi?p=gitmo%2FMoose.git&a=search&st=author&s=Mark+Stosberg

[10]: http://github.com/nothingmuch/moose
[11]: http://search.cpan.org/dist/Moose/lib/Moose/Manual/Contributing.pod
