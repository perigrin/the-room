Title: Version Numbers and Deprecation Policies  
Author: Chris Prather
Date: 2009-08-24 15:43:58

# Version Numbers and Deprecation Policies
A few days mst pointed out to me the Mojo release notes. I don't pay attention
to the project since these days I have my plate full with [Moose][1],
[Catalyst][2], and [KiokuDB][3] as well as starting a small business to be
paying attention to yet another wheel.

My comment was "wow". To quote the release notes

    0.991250 2009-08-18 00:00:00
        - This release contains many substantial changes that are not
          backwards compatible, but good news is that it's also the last
          major feature breaking release before 1.0. ;)
          Older releases of Mojo did contain additional Mojo::Script::* and
          Mojolicious::Script::* modules that are obsolete now and might
          break this version if they are still present on your system.
          Because of this we highly suggest that you
          DELETE ALL MODULES IN THE "Mojo", "MojoX" AND "Mojolicious"
          NAMESPACES MANUALLY!!!

I was shocked that the deprecation policy of a framework that is being touted
so widely as [attractive to non perl people][10] would involve the phrase
"DELETE ALL MODULES". I've been a Perl developer for over a decade now and I'm
not comfortable with having to hunt through my file system and delete files
manually. I can only imagine what someone with less experience dealing with
Perl's &#133; evolved &#133; filesystem would think (The phrase "WTF?!?" comes to mind).

But that said, I happily went on about my business because what Mojo does has
no bearing on my life since I don't use it. Today's feed pulls up [Sebastian's
reply][6] to [mst's blog post][5] about deprecations in which is take away was
that mst doesn't understand version numbers. I think someone is missing the
point.

<blockquote>
    As soon as something hits CPAN it is considered stable, <a href="http://en.wikipedia.org/wiki/Software_versioning">good versioning
    practices</a> are simply thrown overboard. It is pretty much industry standard
    to use "0." versions for code that you want to release for early user
    feedback but thats still in development.
</blockquote>
This may be true, but the very page he links to there doesn't say anything
about industry standards. In fact it documents in some 4,200+ words that there
is no "standard" for versioning. It also says that some projects "use the
major version number to indicate incompatible releases". Which arguably is
*also* an industry standard.

The fact is the only thing you can safely rely upon a version to tell you is
that this release is different from that release by at least one line. The
line that sets the version number. This is why projects need explicit
deprecations policies, which is the real point of mst's post. One that
[zby][8] managed to pick up.

zby picks up from Sebastian's line

<blockquote>
    Should a project have deprecation policies as soon as it hits CPAN with a
    development release?
</blockquote>

and says 
<blockquote>
    I think the answer should be yes - there should always be a deprecation
    policy. But this deprecation policy does not need to be the full monty of
    always keeping the backcompat to at least one major version, using strict
    deprecation cycles etc - it just needs to be stated explicitely and not
    relying on the version numbering convention as they are not really
    universal.
</blockquote>

I think that in truth *every* module (and project) has a deprecation policy &#133;
they're just usually implicit. Users have expectations. They expect that
packages touted as making their life easier *will* in fact make their life
easier. They expect code to be stable and to not blithely cause them
unnecessary pain. They also expect growth and that old features that they
personally don't use will not get in the way of the future because that causes
pain too.

This means that unless you write down explicitly what you plan you not only
have an implicit deprecation policy, you have one for every single person that
uses your code and has different expectations on how the future releases will
affect them. It also means that if you make seemingly random breakages that
require manual deletion of file, or even just break your API in way that
causes pain without some warning you're [gonna get called on it][9].

People are not perfect. Communication really is the only solution. In the
absence of documented authoritative communication users will form their own
opinions.

UPDATE: Apparently TextMate's Markdown syntax and MovableType's are not remotely similar. I have fixed the formatting since Sebastian tweeted about this so now I'm sure people are actually gonna read it.

[1]: http://search.cpan.org/dist/Moose
[2]: http://search.cpan.org/dist/Catalyst-Runtime
[3]: http://search.cpan.org/dist/KiokuDB
[5]: http://www.shadowcat.co.uk/blog/matt-s-trout/backwards-compatibility-and-migration-paths/
[6]: http://labs.kraih.com/blog/2009/08/version-numbers-and-backwards-compatibility.html
[7]: http://en.wikipedia.org/wiki/Software_versioning
[8]: http://perlalchemy.blogspot.com/2009/08/lets-write-some-deprecation-policies.html
[9]: http://use.perl.org/articles/07/12/21/0113225.shtml
[10]: http://labs.kraih.com/blog/2009/08/the-times-they-are-a-changing.html
