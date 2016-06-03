---
layout: post
Title: Brewing Up a Storm
Author: Chris Prather
Date: 2010-03-07
---

# Brewing Up a Storm

Because I recently had the opportunity to do a fresh re-install of my world[^1], I've spent the last few days playing with [`App::perlbrew`][1]. `App::perlbrew` is the invention of Kang-min Liu aka gugod, and the basic idea is that it's a perl manager. It will install and track several different installations of Perl for you, allowing you to switch between them at will.

	$perlbrew installed
	perl-5.10.1
	perl-5.8.9
	perl-5.6.2

	$perlbrew switch perl-5.10.1
	$perl -v
	This is perl, v5.10.1 (*) built for darwin-2level

I'd like to state now that what gugod had was very nice. It was very simple, very straight forward and did exactly what it said on the package. I have not spoke with him about this application, and anything I say here is my opinion and has no reflection upon him or anybody else that might have been involved. 

Having seen what power some simple scripting can do in the form of Miyagawa's `App::cpanminus`, I started tinkering with `perlbrew`. It started with perl 5.10.1 not installing properly on Snow Leopard[^2], so the first thing I added was a way to force install. 

	perlbrew install -f perl-5.10.1

Then I decided it and `local::lib` both sharing $HOME/perl5 wasn't going to be pretty. So I taught it to use an environment variable `PERLBREW_ROOT` to relocate the default install. In my setup I have it set to `$HOME/.perlbrew`.

	$export PERLBREW_ROOT=$HOME/.perlbrew
	$perlbrew init
	Attempting to create directory /Users/perigrin/.perlbrew/perls/current
	Perlbrew environmet Initiated.
	Required directories are created under /Users/perigrin/.perlbrew.
	Please add this to the end of your ~/.bashrc:
	    source /Users/perigrin/.perlbrew/etc/bashrc

Then I got annoyed with the pages of verbose output, so I stole a page from `cpanminus` and implemented a `quiet` switch that is enabled by default. Output now goes to `$PERLBREW_ROOT/build.log`.

Today I integrated it with `local::lib` so that if you have `local::lib` installed it will drop the proper configuration in the `perlbrew` configuration scripts so that when you install new modules they get installed into your `perlbrew` managed directories[^3].

Finally just now I finished getting `--as=` working. This means that if you build a custom perl you can have it installed under a special name and then easily switch to and from it. I'm planning on using this to build a `--as-workperl` that contains all the modules I need for work. 

	$perlbrew install perl-5.10.1 --as=debugging-perl -D=debugging

I would like to integrate a plugin system[^4], but all in all I'm very happy with it. Much thanks to gugod for making it to start with, and to Miyagawa for getting me itching for small lightweight tools in my toolchain. If you're interested in my changes, you can check them out on my [github][2]

[^1]: Last week I upgraded my trusty Macbook Pro from OSX 10.5 (Leopard) to OSX 10.6 (Snow Leopard), using a "complete wipe". The old system had been around for ~3 years and was showing the cruft so it was time.

[^2]: Apparently there is an issue with Snow Leopard's locales, and a single test fails.

[^3]: Full disclosure, I got this to the proof of concept stage but not really much further. It currently expects `local::lib` to be installed to `$HOME/perl5` (which is the default). If you have it installed somewhere else, you'll need to manually set up the environment.

[^4]: Having recently played with plugins in `cpanminus` I have to say they're incredibly nifty. The `github` plugin is especially nice. I especially would like a plugin to allow swapping Git in for several parts of the system.

[1]: http://search.cpan.org/dist/App-perlbrew
[2]: http://github.com/perigrin/App-perlbrew