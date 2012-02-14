Title: I saw the best minds of my generation...
Author: Chris Prather
Date: 2012-02-13 19:05:00


# I saw the best minds of my generation...

So there has been a recent [post][1] that stirred up some commentary. The basic gist is that the author decided to leave Perl because he got tired of living with the constant scolding over 'use strict'. Something that (as he points out in an update) even Mark Jason Dominus has [commented on][2][^1]. 

I agree with him. Not about `use strict`, I actually agree with the people that suggest using more training wheels like strict, warnings, and diagnostics. I think that not only do they help when you're first starting out, they help protect against stupid mistakes when you've been doing this for a decade. No, I agree that the Perl community has managed to make programming less-fun, and I think the comments on his post prove his (and now my) point. 

> The big reason for leaving Perl, though, was people in the Perl community, who had become annoying scolds.

Most of the comments focus on the 'OMG He is complaining about `strict` and then using Python!", without pausing for a moment and looking at the deeper issue. As one commenter points out we use `strict` as a test to identify people who are worthy of our support from those who are un-worthy. We have a lot of these shibboleths, and we can't even agree on them as a community. I've had people file bugs against modules I maintain because they use Module::Install. I've had people file bugs against modules I maintain because they *don't* use Module::Install. I've watched people get rather nasty in blog posts because an author decided to use Moose, or stop using Moose, or use Mouse, or Moo, or Mo, or M ... well maybe not M. 

If we're going to have a reputation for being annoying scolds, we could at least have the decency to come up with an single set of standards to scold people with.

I'd rather however we stop for a minute and remember that there is another developer on the other side of the computer. I've been guilty of ignoring this as much as anybody[^2]. But we're not just losing developers like the author of this article. I've had someone who helped me get started in the Perl community tell me that this article sums up the way he feels. That's two people who have had over a deacde in Perl tell me they moved on because Perl stopped being fun. How many people are there that feel this way and don't say anything?

So here's the deal, the next time you feel you need to complain that someone is using Moose, or not using Moose, or using Dist::Zilla, or using Module::Install ... rather than explaining to them how their decision has personally caused you anguish and has been shown in peer reviewed studies to cause cancer in Kittens, stop. Email them, or message them, or whatever and ask them why they chose that path. They may tell you that they didn't know a better way, that they were just trying something new, or that they need more research candidates for their research studies on curing cancer. 

If you can't accept their opinion at that point, ask yourself what you can do to help people become better informed on what the appropriate opinions are in the future. If you can't come up with something simple you can do, ask yourself why you can't contribute to making the Perl community better. 

Then submit a talk to the next Perl workshop[^3] about why you hate kittens.

[^1]: If you have no clue who MJD is you can skip this blog post because you haven't been around long enough to remember when things were different. If you have no clue why MJD's opinion might possibly matter, well ... thank you mjd for reading my blog.

[^2]: Especially on reddit, if I've been an asshat to you there and haven't apologized ... I'm sorry, for both being an asshat and this crappy apology.

[^3]: At the time of this writing that is the DCBPW Workshop, and/or YAPC::NA 2012. Both of which I believe are accpeting talks.

[1]: http://www.leancrew.com/all-this/2012/02/why-i-left-perl/
[2]: http://perl.plover.com/yak/12views/samples/notes.html#sl-31
