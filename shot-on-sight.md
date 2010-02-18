Title: Why I should (apparently) be shot on sight
Author: Chris Prather
Date: 2010-02-18 03:42

# Why I should (apparently) be shot on sight

Recently `chromatic` posted:

>   [12:52] chromatic: More and more I think Module::Install *hurts* 
>   CPAN's installation experience.  Ia!  Ia! EUMM fhtagn!

Which I found curious. I suspect it is because it resorts to driving ExtUtils::MakeMaker which `chromatic` has been vocal about hating in the past. It was however Elliot Shank's comments in response that I found really off putting.

>   [12:58] clonezone: .@chromatic_x I want to throttle all CPAN authors
>   that use Module::Install. M::I may be great for the author, but it
>   sucks as a user.

>   [12:59] clonezone: Any CPAN author that uses Module::AutoInstall
>   should be shot on sight.

Needless to say I use both [Module::Install][1] and [Module::AutoInstall][2], and before the firing squad shows up to execute my unrepentant self I figured I should explain for posterity's sake.

First some history. In 2003 I became interested in the [OpenGuides](http://openguides.org) project. OpenGuides is a Perl based wiki to build guidebook style websites for Cities. I maintained a guide or two for a while before losing the time and resources to host them.

OpenGuides uses Module::Build. Early on when I was starting to work with OpenGuides as a Guide Admin, a troublesome build of Module::Build had gotten into CPAN. I don't recall the details and to be honest they're not important. Things happen, not every release can be perfect. This build basically caused a ton of pain for anything that used Module::Build for installation. 

Obviously Module::Build has fixed these problems since then. I have not had a problem in over five years with Module::Build. Most of the time I don't recognize that it is even being used[^1]. This story isn't about Module::Build, it's about why I deserve to be shot on sight.

The experience left a bad taste in my mouth. One bad module could break my entire toolchain. A module that the maintainers of OpenGuides didn't control and couldn't fix for me the user. That was a horrible experience, and it came right as I was starting to release my first Perl modules onto CPAN. 

I am lazy, and at the time I was using Module::Starter to build my distributions. Module::Starter would build my Makefile.PL to use EUMM and everything generally "just worked", but I still didn't like the fact that I had to rely upon the user having the right toolchain installed. When someone, I suspect `mst` suggested I take a look at Module::Install because it auto bundled itself into `inc/`. I was curious enough to take a look. The declarative sugar was nice, but the bundling was the important part. 

Now there has been an ages old debate about bundling. Adam Kennedy explained it well enough

>    In the Module::Install model, all authors need to do incremental
>    releases of the modules affected by the problem, but users need to
>    do nothing.
>
>    In the Module::Build model, all users affected by the problem need
>    to upgrade their version of Module::Build, but authors need to do
>    nothing.
>
>    At present, both of these don't solve this problem correctly.
>
>    Module::Install doesn't have a method for ensuring authors upgrade,
>    and Module::Build doesn't have a method for ensuring users upgrade.

I quite obviously lean strongly on the "make the author responsible" side of the fence. The number of Authors of CPAN modules is smaller than the number of Users of CPAN modules. Make the pain point as small as possible is my argument. Module::Build even supports bundling itself now as well. So this story isn't really about bundling either, this story is about why I deserve to be shot on sight.

When one starts using Module::Install one quickly finds Module::AutoInstall, the one that has the firing squad after me. Module::AutoInstall will spawn a CPAN/CPANPLUS instance to chase down dependencies[^2]. We use this at one of my jobs to maintain our dependencies for the very large application we're maintaining, but it's a feature that CPAN and CPANPLUS both have managed to do for quite some time now.

The other feature that Module::AutoInstall performs, the one I'm willing to be shot for, is Features. Features are what make utilities like [`Task::Kensho`][3], [Task::Catalyst][4], and [Task::Moose][5] useful. They allow you to gather, possibly optional, dependencies together into smaller groups and present that as a choice to the user. Features do not exist in ExtUtils::MakeMaker, and in Module::Build the closest I could find was a "reccomends" dependency which lacks the grouping (as far as I could tell from the documentation).

I have three modules that require Features to properly work. `Task::Kensho` and `JSON::Any` are the best known of them. Without Module::AutoInstall these two modules would be a much bigger hassle for my users. `JSON::Any` for example would require me to define a specific default JSON package and make the others "reccomended", this rather defeats the entire purpose of `JSON::Any`. As for `Task::Kensho`, one of the things I like most about it is that beyond a certain set of core modules relating to the tool chain and testing, everything else is optional. This means if you don't want or need [POE](http://search.cpan.org/dist/POE) on your machine (perhaps because you enjoy AnyEvent), `Task::Kensho` doesn't force you to take it. But it does so at a large enough level that if you opt into POE, you get a good set of recommended modules picked by default.

I am also lazy, I mentioned this. I have several Modules that need to be updated to no longer include `auto_install` because they don't need or use Features. But despite at least one of these modules being popular enough to make a Top 100 list, I have never had a complaint about not being able to install it. I cannot repent for thinking about my users[^3]. So I will clean up the modules I have that don't require Module::AutoInstall while I wait for the firing squad for the few modules that *must* have it so that the user experience is the nicest I can provide.

[^1]: I give much thanks to Ken Williams, Michael Schwern, Eric Wilhelm, and David Golden for the work they've done over the years to make sure I don't notice Module::Build anymore. They are awesome people who are making the world better.

[^2]: It used to do this regardless of the environment it was in, but now has checks to bail if it's under a CPAN/CPANPLUS install. They're not perfect checks, but they should work on any CPAN releaed in the last two to three years.

[^3]: If I truly were going to make life easier on myself I would have moved to [Dist::Zilla](http://search.cpan.org/dist/Dist-Zilla) and stopped even writing a Makefile.PL.

[1]: http://search.cpan.org/dist/Module-Install/lib/Module/Install.pm
[2]: http://search.cpan.org/dist/Module-Install/lib/Module/AutoInstall.pm
[3]: http://search.cpan.org/dist/Task-Kensho
[4]: http://search.cpan.org/dist/Task-Catalyst
[5]: http://search.cpan.org/dist/Task-Moose