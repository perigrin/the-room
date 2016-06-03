---
layout: post
Title: Ring Ring Ring Ring Perl Phone!  
Author: Chris Prather
Date: 2009-09-05 15:42:11
---

# Ring Ring Ring Ring Perl Phone!
So I just got my first Perl script running on my Phone. I have a T-Mobile G1 and I installed the [Android Scripting Environment 0.11-Alpha][2] which allows you to install a copy of Perl5 (as of this writing 5.10.0) on your phone. It then sets up an RPC enviroment giving you access to the native Android APIs via a very naieve JSON-RPC system.

There was a small bug in the JSON-RPC Code, but it was [easily fixed][1]. Now my [simple script][3] test script works as expected. DAHUT!

UPDATE: Apparently the XS side of things isn't working right, which means that any module (including core Modules) that requires XS doesn't work. Also some non-XS core modules (like Time::Local) apparently aren't 
included by default either. This is causing LWP to not work properly. Hopefully someone with better C skills out there can help the cross compilation of XS modules.

[1]: http://github.com/perigrin/android-scripting-environment-perl/commit/f503d9e89fec81009a208087df7b94f791baba1b
[2]: http://code.google.com/p/android-scripting/
[3]: http://github.com/perigrin/android-scripting-environment-perl/blob/f503d9e89fec81009a208087df7b94f791baba1b/scripts/dahut.pl
