---
layout: post
Title: Change Management is *Hard*  
Author: Chris Prather
Date: 2009-07-02 17:26:06
---

# Change Management is *Hard*
So I've just release a new [JSON::Any][1]. Back in January when I made the last release, Marc Mims (the new maint for [Net::Twitter][2]) had a bug in JSON::Syck which Audrey decided not to fix since JSON::Syck is deprecated. This is fair, she's the maintainer and is allowed to deprecate things when there are better alternatives on CPAN (which honestly [JSON][3] / [JSON::XS][4] are).

JSON::Any has a feature where you can explicitly load a list of modules by passing them in 
    
    use JSON::Any qw(XS Syck DWIW)

and a default list that will try to *implicitly* load modules. In the 1.19 release I removed JSON::Syck from the implicit list entirely. This meant that you'd have to explicitly pass it in if you wanted to optionally use JSON::Syck. This sounded reasonable at the time, but turned out to be a poor decision. If all you have is JSON::Syck installed, then JSON::Any is useless. This turned up in a bug to [Test::JSON][5].

So JSON::Any 1.20 is on it's way to CPAN, and it now includes a Deprecation feature where modules that JSON::Any supports but we don't want to blindly trust are loaded via a deprecation list with a warning. This is a subtle prod to entice people to upgrade. Currently the warning is non-optional but I'm formulating different ideas on how to enable/disable the warning in different contexts, but like anything with JSON::Any it may not happen until someone complains loudly ... or with a patch.

[1]: http://search.cpan.org/dist/json-any
[2]: http://search.cpan.org/dist/net-twitter
[3]: http://search.cpan.org/dist/json
[4]: http://search.cpan.org/dist/json-xs
[5]: http://search.cpan.org/dist/test-json
