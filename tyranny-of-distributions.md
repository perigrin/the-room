Date: 2011-05-08T12:39:00

# The Tyranny of Distributions

The Mojolicious team has deprecated Mojolicious on Perl 5.8. Sebastian has a really [good argument][1] for doing this, 5.8's regular expression engine is causing security problems. Considering that Perl 5.8 has been deprecated by the core Perl developers, and that 5.10 will be deprecated when Perl 5.14 is released sometime very soon, dropping support for 5.8 really shouldn't be very controversial.

So it's no surprised that he has seen a lot of push-back. The most eloquent critic has been [rassie][2]. The point is that some distributions (notably RedHat Enterprise Linux, and CentOS that mirrors it) haven't yet migrated off of 5.8. 

> A lot of Perl code runs in closed corporate environments and thus has to obey some rules, like 
> using special software distributions, so [unsurprisingly], a big chunk of those 5.8ers are 
> enterprise Linux distributions like RHEL or SLES. 

The standard response to this is incredulity at using the system Perl for anything. While I fully agree that relying upon the system Perl is itself a security risk[^1], the solution of "Just install perlbrew" leaves a lot lacking. Again rassie states it elegantly:

> Nobody in their right mind would have allowed perlbrew on production servers. Nobody in their 
> right mind would allow me to allocate hours and days of developers’ and testers’ time to 
> retest everything with the newest Perl version (you don’t seriously assume legacy code had any 
> amount of reasonable regression tests?)

The standard answer again is to blame the distributions. I know I have complained but as rassie points out modern Perl version simply weren't there when RHEL5 was released. 

> Perl 5.8 has been released in 2002 and as you see, RHEL3, released a year later, has [caught] 
> up and delivered 5.8. RHEL4 and RHEL5 have seen minor Perl updates, since no major updates for 
> Perl came out in that time. Perl 5.10 appeared half a year after RHEL5 released, so it hasn’t 
> been included, Perl 5.12 has appeared roundabout at the same time as RHEL6 beta, so only 5.10 
> could have been shipped.

But this obscures the real problem. One that Sebastian points out in the email thread about deprecating Mojolicious on 5.8.

> Perl 5.8 is no longer supported by the community, RedHat will keep it 
> compiling on their platform but nothing more, these bugs will never 
> get fixed in the 5.8 family. (http://rt.perl.org/rt3//Public/Bug/Display.html?id=49956) 

It's wonderful that RedHat has a 7 year support license for RedHat Enterprise Linux. That means absolutely nothing for Perl. RedHat doesn't promise to even maintain security patches for Perl for 7 years. If your company depends upon Perl, the RedHat support license is only half the story. 

So what then is the answer? Well the best answer would be for the companies making promises of 7 years of support to actually provide support for those 7 years and to step up and make sure that security patches and what not are applied to Perl 5.8. That isn't going to happen.

An answer could be that firms that depend upon Perl realize that Perl has a separate support license from the Operating System and to employ someone, or hire a company who offers a support license for Perl to maintain Perl 5.8 for however long they need. That probably won't happen either.

The answer is not to blame people who volunteer their time for not being mindful of your distribution environment. To be frank that isn't their problem. The guilt/feel-good motivations of Open Source only work so far. I personally am tempted to drop support for any unsupported Perl version for my own modules (and to recommend to the communities, and clients I work with to do the same). None of those communities has the ability to support deprecated versions of Perl independently.  My company certainly doesn't employ anybody with sufficient knowledge to contribute to the *current* version of Perl, let alone be able to maintain a legacy version of Perl single handedly. So what are we left with? 


We do what the Perl core team does (under Jesse Vincent's admirable leadership). We try to maintain compatibility as best we can, while trying to keep things moving forward to be more useful, secure, and robust. This is what Sebastian is doing with Mojolicious[^2]. 

If you have a need to support a legacy version of any of my modules, I am wildly supportive of you forking that to maintain compatibility with your environment. I don't work in your environment, I'm not motivated by guilt (or money) enough to pretend that I do. 


[^1]:Apple, for example, has consistently broken their own system Perl on OSX making using it for even development unreliable. The current Perl in Snow Leopard in fact cannot build XS modules using the current release of XCode4 without tweaking the environment in an obscure way. 

[^2]: Don't believe it is pure altruism. Sebastian has much snarkier reasons. "Just to make it absolutely clear, [I] have no intention to give other Perl web frameworks a competitive advantage." 

[1]: http://groups.google.com/group/mojolicious/browse_thread/thread/510dcf2219371deb?pli=1
[2]: http://rassie.org/archives/378