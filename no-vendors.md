Title: No Vendors?
Author: Chris Prather
Date: 2010-03-17

# No Vendors?

Recently in a post about his [moving to Google][1],  Tim Bray said:

>   The big thing about the Web isn’t the technology, it’s that it’s the
>   first-ever platform without a vendor (credit for first pointing this
>   out goes to Dave Winer).

I've always respected Mr. Bray, I follow his blog avidly and I really enjoyed his discussion of learning the Android platform and his Wide Finder project. While I may think that his Ruby fanboy-isim is a bit over the top, I'm unabashedly in love with Modern Perl so who am I to really talk?

This comment though, about the web having no vendor's. I think the point he was trying to make is that Tim Berners-Lee released the technology for the web to the public domain (not GPL, MIT, Artistc or any other licese but ___Public Domain___) and thus anybody and everybody is free to implement the specifications without fear that someone will come along and require a license fee.

However that's not the only thing a Vendor does. A vendor supplies a technology or tool, documentation for using that tool, and support for when bugs are found in that tool. The "Web" in Mr. Bray's view may only be HTTP, HTML, and the related W3C specifications, but ultimately it's much more than that.  The web has mutated over the last two decades. Today it isn't just HTTP and HTML, it's also FBML, and REST Apis, and Twitter.

This brings me to the [recent post][2] by John Napiorkowski, a fellow Perl programmer and Moose developer. John pointed out that the Google Adwords API is introducing backwards incompatible changes, and that the current Perl API is going to be broken by these changes. This is exactly Google acting as a Vendor for a SaaS product, one that many people rely upon for income.

Google's [official statement][3] regarding support for the Perl API?

>   We recommend that developers either migrate their applications to
>   another language and client library (such as PHP, Python, etc.) or 	
>   continue the development of their own implementation in Perl.

That's google acting like a *crappy* vendor. This recommendation is an insult to your customers. Google is saying that it would be easier if people spent the next month re-writing what may be a core portion of their business (and if you're using adwords is not *not* a core part of your business?) in a brand new language, rather than depend upon Google to care enough to bring one of their ___vendor supplied___ libraries up to date.

Now in Google's defense they prefaced that recommendation with:

>   We are aware that the Perl client library is out of date, and we are 
>   actively working on updating it to take full advantage of the 
>   v200909 version of the API.  We encourage AdWords API Perl 
>   developers to contribute to the open source project and help to 
>   accelerate the process, but we want you to understand that this work 
>   will not be completed prior to the April 22 sunset of most v13 
>   services.

Thank you Google for saying that even though you don't care enough about Perl to support your *paying* customers you care enough about open source to let those same customers do the work themselves. This is a step forward from the way things were in my Mom's day. 

Google may not be evil, but they're certainly being crappy.

[1]: http://www.tbray.org/ongoing/When/201x/2010/03/15/Joining-Google
[2]: http://jjnapiorkowski.vox.com/library/post/google-do-no-evil-to-perl.html
[3]: http://groups.google.com/group/adwords-api/browse_thread/thread/d738463da3b5bdbe?pli=1