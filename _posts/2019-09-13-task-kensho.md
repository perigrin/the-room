---
layout: post
title: "Task::Kensho at 11"
author: "Chris Prather"
categories:
tags: [perl]
date:  2019-09-14T13:04:36-0500
image: https://live.staticflickr.com/4509/37561404372_1f14057b3e_z.jpg
---


So Grinnz wrote [a blog post](https://blogs.perl.org/users/grinnz/2019/09/taskkensho-needs-your-help.html)
calling for help with Task::Kensho. This was something I wanted to do myself
recently but life happened. So I'm super greatful he stepped up and did it.
There are I think several really good suggestions in the issues queue that need
some more discussion to them.

In [the reddit thread](https://www.reddit.com/r/perl/comments/d38pw7/taskkensho_needs_your_help/)
that spawned from his blog post there were some comments that I started to
reply to there but then realized that it was going from a few comments into
someting much more involved.

First the name, as someone pointed out Kensho isn't exactly an obvious idea,
even for (perahps especially for) native Japanese speakers. Which is fair, but
as Grinnz points out:

> It also seems a bit more hubris to upload something called
> Task::RecommendedModules.

More hubris than the original author has to this day. Really if we were being
descriptive of the original intent Task::StopMstFromWhinging would have been
it's name, and that wouldn't have been any better.

From a historical perspective Task::Kensho came from the time when modules were
being named things like Moose, Catalyst, and Starman. Specifically it predates
the "awesome" github repositories that were all the rage for a while
(specifically [awesome-perl](https://github.com/hachiojipm/awesome-perl)[^1] It
probably would have been that had those been a thing when Task::Kensho was
written. Which sorta is a theme in Perl space. A lot of our weirdness is
because we tried things before they were popular elsewhere. And the elsewheres
never bothered to ask us our opinions when they became popular.

It came at a time when Task:: modules were being created. It was originally a
default install of clear choices. But slowly as time went on people recommended
(and rightly so) mutually alternative choices, and we didn't have a space to
discuss which choice should be the *default*. Maybe we have that space now in
the Github issues.

Second, I like having Task::Kensho on CPAN because CPAN is Perl. Other
languages have embraced github in beautiful ways (I'm looking at you go with
`import "github.com/foo/bar"`), and that's great for them. But CPAN is where
Perl does it's thing, for better or worse[^2]. I like having Task::Kensho just
be something you just run with `cpanm -i Task::Kensho` and start making the
choices.

Finally, I discovered recently that Task::Kensho just turned 11 years old. I
mean I knew it started sometime in 2007/2008 but I hadn't really thought about
what that meant chronologically. For a time it was being reccomended on the
front page of metacpan (until Febuary of this year even). For something that
started out as a quick and dirty hack project to stop someone complaining that
there wasn't anything like it out there that's kinda awesome.

---

The title image is [Degustación Kensho Mediterranean Sake](https://flic.kr/p/Zeb1Ud)
by [Mistral Bonsai](https://www.flickr.com/photos/mistralbonsai/) on Flickr.

[^1]: The first CPAN release of Task::Kensho is only 7 months after Github was founded according to Wikipedia.

[^2]: Trust me I have my complaints about CPAN, and Perl modules, and our tool-chain. But they're things that we as Perl developers have to live with. I have more respect for the languages that explain their idiosyncratic tool-chains than the ones that try to protect me, the newbie, from their crazy legacy choices.
