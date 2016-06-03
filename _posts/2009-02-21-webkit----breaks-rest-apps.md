---
layout: post
Title: webkit-- # breaks REST apps  
Author: Chris Prather
Date: 2009-02-21 18:40:28
---

# webkit-- # breaks REST apps
WebKit is has a broken Accept header, it puts text/xml and application/xml
first which breaks Catalyst::Action::REST's default configuration and makes
the idea of being able to dispatch html/xhtml different from XML difficult at
best.

The reason it turns out is that the webkit developers [cargo-culted from
Firefox][cargo-cult], and someone in 2007 provided a [patch][patch]. The
response to the patch is "buh buh Firefox is doing it!". When I tried to
add the comment below I discovered that their bugzilla required me to log in
and didn't appear to have an option for OpenID. Because I'm loath to create
*yet another* account to file a single comment on a bug I've included the text
here:

> Firefox (3.1 beta 2 at least) is no longer sending this Accept headers.
> 
> Currently writing a REST-ful interface that renders XML different from
> XHTML/HTML at the same URI is difficult (requires us to browser sniff and do
> we really want to go back to that?) because Webkit based browsers will all
> send then wrong Accept headers first. I actually ran into this while writing
> an app that targets Android/iPhone and I had to disable my XML rendering code
> (luckily it was a stub and not a required feature).
> 
> Last the logic "Firefox does this" is a fallacy that I thought most people
> were taught better by their mothers at an early age ... if Firefox were to
> jump off a bridge should WebKit too?
> 

[cargo-cult]: https://bugs.webkit.org/show_bug.cgi?id=9572
[patch]: https://bugs.webkit.org/show_bug.cgi?id=12296

