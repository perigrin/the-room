Title: How to Break CPAN with Dist::Zilla!
Author: Chris Prather
Date: Tue Apr  3 13:22:22 EDT 2012

---

# How to Automate breaking half of CPAN with Dist::Zilla!

So last night I dropped a [new module][crixa] on to the CPAN. Normally this would be a cause for celelbration, except it turns out I broke Moose.

## A Date with Destiny

If you don't know [local::lib][local-lib], you should. I've been using it on a per-project level using John Napiorkowski's [App::local::lib::helper][localenv]. This works out incredibly well for those nasty internal projects that have hard dependencies on specific versions of modules, or have dependencies on modules that don't want to talk to each other.

I've been using this so much I have a couple scripts setup to make life easier for myself, I just run `use-local-lib` and it'll setup a `perl5/` directory and setup a subshell to make that the default installation target for things like `cpanm`.

And because I've been incredibly busy [learning new things][yapc] that are totally unrelated to being a developer, `Dist::Zilla` has been helping me by making sure things happen the same way every time regardless of how long it's been since I'd done release engineering. It helps prevent issues like [this][rt]. 

## With Great Automation comes Great Responsibility

So the stage was set last night. I'd decided to cut a release of my latest module that I'd been working on for a client project so that I could actually *use* it for that client project. Just fire up `dzil release`, fix the several issues that cropped up with the Git plugin and get it to ship off the latest dist.

*Then* I notice that it had shipped my `local::lib`. 

This was made worse by the fact that I happen to have co-maint on a lot of modules that I used on a daily basis. Not that I *wrote* all of them, just that people (until today) thought I was responsible enough to be a bus number for them. So I had co-maint on large portions of my `local::lib`.

Crap.

## The Fix

The main fix (and the one I didn't do in a timely enough manner) was to run crying to my local PAUSE admin and tell them what I'd done. 

What I had done was immediately when I noticed issue a delete request for the bad dist, and create a new good dist and upload that. What needed to happen was for a PAUSE admin to force a reindex (and even then `doy` had to create a new release of Moose) of the affected modules. 

Finally, I just needed to teach dzil to prune the perl5 directory so that it never happened again. Adding the following to my `dist.ini` fixed it.

    [PruneFiles]
    match = ^perl5

This will make sure that after dzil creates the build directory, it removes the local::lib it copied over from the source directory. 

## Knowing is Half the Battle

So Dist::Zilla is a huge boon not just to people who manage *very large numbers* of CPAN dists, it's also nice for people who in-frequently manage CPAN distributions. But with the great power of automation, comes great responsibilty to make sure that it does what you're expecting. I *should* have taken a minute or two to check that the distribution direction was what I had expected it to be, and when things had blown up I *should* have gone and talked to someone to make sure that it had been cleaned up properly.


[crixa]: https://metacpan.org/module/Crixa
[local-lib]: https://metacpan.org/module/local::lib
[yapc]: http://act.yapcna.org/2012/talk/39 
[rt]: https://rt.cpan.org/Ticket/Display.html?id=76225
[localenv]: https://metacpan.org/module/App::local::lib::helper
