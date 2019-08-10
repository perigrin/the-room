---
layout: post
Author: Chris Prather
Title: Future Moose Support
Date: 2011-05-20 12:50:49
---

---

There has been a little bit of noise on twitter about the recent policy decision by the Moose core team. Most of it seems to focus on this line:

> Moose will be dropping support for perl 5.8 with the 2.06 release (in January of 2012).

Which is stating that nearly a year after support for 5.10 has been dropped by the core perl development team (i.e. perl5-porters), Moose will be dropping support for the version *before* that. A version that hasn't been supported by perl5-porters since 5.12.0 was released in April 2010, and which hasn't seen a release since December 2008.

If that was all that Jesse had said in the announcement, I would totally understand the tempest in a teapot. Mojolicious recently went through the same reaction, and I've commented on it [previously][1]. It isn't, and people seem to be ignoring the entire paragraph that follows that first sentence.

> As for what this actually means: we will not be immediately rushing in to start using smart
> matching and ripping out our existing 5.8 compatibility code. The specified version requirement
> will stay at 5.8.3. Moose will likely continue to work on perl 5.8 for a good while after next
> January. What this does mean, however, is that we will not be spending any more time fixing 5.8-
> specific issues. If critical bugs are found (unlikely at this point, but still possible), we
> will likely just bump our version requirement if no patch is forthcoming. New features are a
> fuzzier topic, but if someone comes along and implements a new Moose feature that requires
> features in perl 5.10, that patch will likely be accepted.

This means that the of current Moose core team, all 9 of us, noone has a vested interest in maintaining 5.8 compatibility[^1]. It means that if you have a vested interest on Moose working on 5.8, that you will need to step up and help support Moose working on 5.8. It means that if nobody from the community steps up to help out that we'll have assume that the interest in supporting Moose on 5.8 is not a priority.

It also means that if people propose feature requests that utilize 5.10 specific code, and nobody steps up to back-port the feature we'll assume that 5.8 support is not a priority and apply the feature patch.

This is as Jesse has [said][2], how Open Source works.

[^1]: Now I own a [consulting company](http://tamarou.com), and I would *love* to have a vested interest in fixing core Moose bugs. I'm pretty sure that Jesse and Dave would be open to having the same happen as well. If this is something that there is interest in, please let me know we can work out some kind of Moose Extended Support License.

[1]: http://chris.prather.org/tyranny-of-distributions.md.html
[2]: http://twitter.com/#!/doyster/statuses/71255246382440448
