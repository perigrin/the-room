---
layout: post
Title: What Stops Me From Using Perl6 Today
Author: Chris Prather
Date: 2009-11-16 14:12
---

# What Stops Me From Using Perl6 Today

I have to confess `mst` asked me to step forward and make a comment about the [Perl5][mst] & [Perl6][masak]. I suspect because I'm a prime example of a person who's deeply involved in what `masak` calls the Perl5 story. This is because I've been bound up tightly with the Moose community for a while, and I've recently founded a [company][tamarou] that is doing Modern Enlightened Perl development. My bread is buttered by Perl5.

Recently because of `mst` and `masak` sparking a conversation I have started taking a really close look at Perl6. One of the things that struck me was how polished and complete the core Perl6 language was, and the Rakudo implementation.

This post started out as a much more whiney response to `bakkushan`'s [post][bakkushan] of the same name. For that version of this post I set out to compare Perl6, specifically Rakudo, to my own cherished Moose. The Example object I came up with is a stripped down version of [`Blawd::Entry::MultiMarkdown`][bemm]. 

    use 5.10.0;
    use MooseX::Declare;

    class Blawd::Entry {
        use MooseX::Types::DateTimeX qw(DateTime);
        has [qw(title author content filename)] => (
            isa      => 'Str',
            is       => 'ro',
            required => 1,
        );
        has date => (
            isa      => DateTime,
            is       => 'ro',
            coerce   => 1,
            required => 1,
        );
        has headers => ( isa => 'Str', is => 'ro', default => '' );
        has commit => ( is => 'ro', required => 1 );
    }
    

The Perl6 version is pretty straight forward.

    use v6;
    
    class Blawd::Entry {
        use Temporal;

        has Str $.title     = die "Str title Required";
        has Str $.author    = die "Str author Required";
        has Str $.content   = die "Str content Required";
        has Str $.filename  = die "Str filename Required";
        has DateTime $.date = die "DateTime date required";
        has Object $.commit = die "Object commit required";
        has Str $.headers   = '';
    }


The Perl6 version is shorter, and arguably more elegant. `TimToady` helped me figure out the Perl6 equivalent to Moose's `required => 1`. You just set a default that will throw an exception when evaluated.

But I've lost the type coercion on DateTime. In the Moose version it is provided in the MooseX::Types::DateTimeX library and enabled by the `isa => DateTime, coerce => 1` parameters passed into the `date` attribute constructor. I know this can be done in Perl6 and after several furtive attempts to find it in the documentation as well as a question that `TimToady` answered on `#perl6` I finally found a solution.

    class Int is also { method Time { Time.gmtime(self) } };

Trying to shoe horn that in after a second beer at 1:30am simply wasn't producing the desired results[^*]. Think about that for a second, my complaint was I couldn't get the Type Coercions to work as elegantly as I wanted. That's a far cry from "half finished vaporware".

So while I have reservations about some of the current design regarding the metaobject protocol (I have offered and am still in dialog with the Perl6 people about helping them steal from Moose), Perl6 is a much more complete picture than the general grousing in the Perl5 community would give you.

Does that mean I've jumped ship and will start hacking in Perl6? Probably not. While Perl6 may have more polish than I expected, and is a project that is *well* on it's way to being complete, it lacks some fundamental things I love about Perl5. The biggest of which is CPAN.

Going back to my Blawd app as an example. Blawd is only 814 lines of code because I leverage CPAN. If you look at the dependencies from the `Makefile.PL` you'll see a tight list.

    requires 'aliased';
    requires 'Bread::Board';
    requires 'DateTime';
    requires 'DBI';
    requires 'Git::PurePerl';
    requires 'HTTP::Engine';
    requires 'Moose' => '0.92';
    requires 'MooseX::Aliases';
    requires 'MooseX::Types::DateTime';
    requires 'MooseX::Types::DateTimeX';
    requires 'MooseX::Types::Path::Class';
    requires 'namespace::autoclean';
    requires 'Path::Class';
    requires 'Text::MultiMarkdown' => '1.0.30';
    requires 'Try::Tiny';
    requires 'XML::RSS';

Some of these are hard requirements, `Git::PurePerl` for example. Some of these are soft requirements, `DBI` for example I only use in the MoveableType exporter, and `HTTP::Engine` is only there for the experimental `server` back-end.

The point however is that I have 15 dependencies for code I didn't have to write. If I wanted to start hacking up a Blawd in Perl6 I'd have to port `Git::PurePerl`, which itself turn requires 11 modules.

    'Archive::Extract'           => '0',
    'Compress::Raw::Zlib'        => '0',
    'Compress::Zlib'             => '0',
    'Data::Stream::Bulk'         => '0',
    'DateTime'                   => '0',
    'Digest::SHA1'               => '0',
    'File::Find::Rule'           => '0',
    'IO::Digest',                => '0',
    'Moose'                      => '0',
    'MooseX::StrictConstructor'  => '0',
    'MooseX::Types::Path::Class' => '0',

Now Perl6 doesn't require some of these. Moose will hopefully not be required in Perl6 in the not too distant future. But what about `Digest::SHA1`? A quick google didn't find anything suitable as a replacement, so I'd have to re-write that as well. What it boils down to is there are far more Yaks than I want to shave in the Perl6 ecosystem. Just to replace my little 814 line hack, I'd have to write a Git implementation, a MultiMarkdown renderer, and an RSS feed generator.

I've added `#perl6` to my IRC client. I'm keeping a closer eye on it, but for now I say long live Perl5, further up and further in.


[^*]: I was testing the results of this article with the following simple test.

    bin/perl6 -e'
        class Int is also { method Time { Time.gmtime(self) } };  
        class A { use Temporal; has Time $.foo = die "Not Declared" }; 
        say A.new(foo => 1)
    ' 
    Assignment type check failed; expected Time, but got Int

[mst]: http://www.shadowcat.co.uk/blog/matt-s-trout/f_ck-perl-6/
[masak]: http://use.perl.org/~masak/journal/39912
[tamarou]: http://tamarou.com
[bakkushan]: http://howcaniexplainthis.blogspot.com/2009/11/what-stops-me-from-using-perl-6-today.html
[bemm]: http://github.com/perigrin/blawd/blob/master/lib/Blawd/Entry/MultiMarkdown.pm
