---
Title: The XML::Toolkit Story: Part 1 - The Early Years  
Author: Chris Prather
Date: 2009-05-11 22:45:19
---

# The XML::Toolkit Story: Part 1 - The Early Years
So one of the talks I had accepted to [YAPC][1] is on [XML::Toolkit][2], and I
thought I'd give a bit of a preview of what it is.

XML::Toolkit was created for a project that required de-serializing XML in to
Perl data, exposing that data to be edited, and then re-serializing it so a
Flash client could read it. Because of assumptions made by the Flash script,
child order was vital, so the XML had to not only round-trip semantically, it
had to round-trip structurally. The second fun fact was there was no DTD and
no schema. Some of the XML wouldn't even parse because it wasn't well formed,
but some work with the client sorted that.

So hard requirements:

1) Round-Trip XML 
2) Guess the Schema

Add to this some soft requirements:

1. Use Moose (we were a nearly 100% Moose company)
2. Be fast
3. Do XML properly (ie Not XML::Simple / XML::Smart which IMHO "fake it"**)

So I dipped into some latent XML knowledge. I've been hanging out with the
AxKit people since I started with Perl, and while I haven't done a ton of XML
work in the last 8 years, I have soaked up a few things. Particularly the
knowledge of Kip Hampton's [Perl/XML][3] column from 2002. I knew that I
wanted something that would perform adequately and that I wanted something
flexible, so I reached for the SAX tools on CPAN. I knew that you could write
[drivers for non-sax data][4], and I assumed I could do the inverse. 

After some hacking I came up with a sax parser that would build [Moose Meta
Classes from SAX streams][5]. A little extra tweaking and I had something
useable for the work project. I will try to get a second post up that details
the second major project I had for XML::Toolkit which caused me to clean it up
and release it.

**&nbsp;Sometimes faking it is good enough.

[1]: http://yapc10.org
[2]: http://search.cpan.org/dist/XML-Toolkit/
[3]: http://www.xml.com/pub/at/15
[4]: http://www.xml.com/pub/a/2001/09/19/sax-non-xml-data.html
[5]: http://github.com/perigrin/xml-toolkit/blob/master/lib/XML/Filter/Moose/Class.pm
