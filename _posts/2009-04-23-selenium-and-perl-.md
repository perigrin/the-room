---
layout: post
Title: Selenium and Perl   
Author: Chris Prather
Date: 2009-04-23 00:00:00
---

# Selenium and Perl 
This came across the [Perl Signals][signals] recently:

> We are using Selenium RC for our automation. We are using a combination of
> Selenium RC + Perl. Could somebody please tell me how to generate a html
> report for our Perl tests using Selenium? Any pointers in this regard
> would be appreciated. -- <a href="http://clearspace.openqa.org/message/60971">Shaguftaa</a>

Because this is something we've talked about at work and I have a friend [Mike
Nachbaur][nacho] who has done extensive work with Perl and Selenium for automated
testing, I asked him about it.

> Well, basically people who're doing testing directly with selenium are
> stupid. A better approach is to do testing via <a
> href="http://search.cpan.org/dist/Test-WWW-Selenium/">Test::WWW::Selenium</a>
> instead, so that way you output TAP, and then you can use standard Perl
> test tools to generate reports from your TAP

He also should have mentioned his own [Test::A8N][a8n] which does "Storytest
Automation", which is a fancy way of saying automated acceptance tests.

[nacho]: http://blog.nachbaur.com/
[a8n]: http://search.cpan.org/dist/Test-A8N/
[signals]:http://perl-signals.tumblr.com/
