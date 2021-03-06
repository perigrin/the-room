---
layout: post
Title: Make CPAN Your Bitch  
Author: Chris Prather
Date: 2008-10-17 15:44:39
---

# Make CPAN Your Bitch
So in a comment on Perlbuzz someone asked if this advice given in the [last post][1] would cause the latest (and possibly untested) version of a module to get installed off CPAN. The short answer is "yes". But that only means "yes if it still passes tests" and hopefully the module maintainer has backwards compatibility tests, the better CPAN modules do. 

The long answer is "yes, but you can control the value of CPAN". There are [several][2] [articles][3] [about][4] [private cpan distributions][5]. Basically the idea is you create a mini-CPAN and point the CPAN.pm you use to install your private modules at that. This means that you can lock certain distributions at certain version, while allowing others to upgrade gracefully. It also means that you can use the end to end tools (like cpan2RPM for those of you trapped in RPM hell).

One of the neat knock on effects that I haven't seen from these ideas (and they're not new ideas) is that you could build a smoking environment using the exact same tools that the CPAN Testers use. Who can beat getting a large section of a QA division for free? I know I'm gonna recommend this at work.

[1]: http://chris.prather.org/archives/perl/write-it-like-you-mean-it/
[2]: http://www.slideshare.net/brian_d_foy/mycpan-frozen-perl-lightning-talk
[3]: http://perlbuzz.com/mechanix/2008/01/make-your-own-mini-cpan.html
[4]: http://www.stonehenge.com/merlyn/LinuxMag/col42.html
[5]: http://www.ddj.org/web-development/184416190?pgno=1
