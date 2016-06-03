---
Title: Thumb-Rate.com: A Modern Perl iPhone Application  
Author: Chris Prather
Date: 2009-05-01 23:08:59
---

# Thumb-Rate.com: A Modern Perl iPhone Application
Yuval has been [posting][1] [several][2] [entries][3] about [KiokuDB][4] and I
thought I'd give a chance to talk about how we had used for an application
currently on sale in the [iPhone store][5]. 

[Thumb-Rate][6] is a social app that allows you to simply vote up or down on
things in real life. It's not a complicated, and the source code clocks in at
3791 lines of perl including the Makefile.PL. The basic architecture is a
classic three tier webapp, Catalyst, TT2, and a object store. In addition to
this because we are hoping to have to handle massive throngs of users (calling
all THRONGS!), we have setup a work queue to offline the database writes, and
a memcached instance to serve data from the front end.

The big places we found easy wins were if you're going to use a simple
persistant object storage, Kioku basically defines that. We have had no
significant technical issues. There are some hiccups because Kioku is a new
project and some of the moving pieces need polish (Search::GIN made one of our
developers want to Find::VODKA). The biggest debate really is are you going to
need to do [OLTP or OLAP][3], your storage choice should reflect your needs.

We also used [Catalyst::Controller::REST][7] to feed both the iphone client
(via JSON) and the web front end (via Rendered TT2). There were some more
hiccups here, but at this point I was no longer the primary developer. The one
bit of pain I recall from this is that [WebKit is broken for XML][8].

I'm happy with the toolset we've pulled together and I think that everybody
involved will agree that it's no worse than your typically web application,
and in some places it is a solid improvement.

[1]: http://blog.woobling.org/2009/05/kiokudb.html
[2]: http://blog.woobling.org/2009/05/using-kiokudb-in-catalyst-applications.html
[3]: http://blog.woobling.org/2009/05/oltp-vs-reporting.html
[4]: http://search.cpan.org/dist/KiokuDB/
[5]: http://itunes.apple.com/WebObjects/MZStore.woa/wa/viewSoftware?id=310098205&mt=8
[6]: http://www.thumb-rate.com/
[7]: http://search.cpan.org/dist/Catalyst-Controller-REST/
[8]: http://chris.prather.org/webkit-breaks-rest-apps/
