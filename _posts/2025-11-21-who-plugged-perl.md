---
layout: post
title: "Who P-P-P-Plugged Perl?"
author: "Chris Prather"
tags: perl, history
date: 2025-11-21
image: https://images.unsplash.com/photo-1744441800604-9d8dcb06c291?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

The movie _Who Framed Roger Rabbit_ is based on a deeply weird novel by
Gary K. Wolf in which Roger Rabbit actually is a murderer (there are
several) and is killed halfway through the book—which doesn't stop him,
he is a toon after all. Reading [What Killed
Perl?](https://entropicthoughts.com/what-killed-perl) kind of reminds me
of that. There are a bunch of stories people believe are true, but
looking into what actually happened, those stories are mostly just
that... stories.

I've spent my career in the Perl community, and the story I witnessed is
different than the ones we tell.

The Rise and Fall of Perl is a much more complicated story, where one of
the main characters is Perl itself holding a smoking gun.

## The Phantom Menace

To the outside world it looked like Perl stopped for 19 years to wait for Perl
6. The reality, however, is very different.

Perl 6 was first announced at OSCON in July of 2000. Perl 5.6.0 had been
released in March, and 5.8.0 would be released almost exactly two years later
in July of 2002. Perl uses an even/odd release cycle where even numbers
represent stable releases and odd numbers represent development releases. So
5.7.0, which would become 5.8.0, was first released in September of 2000.

Perl's release cycle at the time was driven by the heroic release engineer
model. The Pumpking—what the Perl community called the release manager—was more
of a lead engineer than simply a release manager. The Pumpking was deeply
involved in the technical aspects of the language development and would often
push out a release when they thought it was "done," having contributed a lot of
the work themselves.

5.9.0 started in October of 2003.

Perl 6 was being designed concurrently while these releases were happening. It
had no apparent effect—releases were still happening.

Behind the scenes several things were going on, and I'm not entirely privy to
all the details. The 5.9.x release series was troubled. 5.9.1 in March 2004.
5.9.2, 5.9.3, 5.9.4 trickling out over the next two years. Finally 5.10.0 in
December 2007. Meanwhile, a different release engineer was steadily releasing
patch releases for 5.8.x during this same period.

Perl 6 has some blame here—how could it not? But I think a far larger portion
of blame goes to resting the entire release process on the shoulders of a
single volunteer. 5.10.0 would be the last major stable release under this
development model. It simply burned people out.

But that's only 7 years, not 19.

## The Renaissance Where Nobody Showed

Here's what was happening from 2010-2019 that everyone outside the Perl
community missed: Perl was completely transforming its development model. We
went from a hero-driven model where a single release engineer would heroically
push out a release when they decided it was "done," to a sustainable,
time-boxed release process.

In 2009, Jesse Vincent had an idea: Pumpking as release manager not release
engineer. Time-boxed releases: monthly dev releases, annual stable releases. It
looked like salvation.

Starting with 5.13, development releases happened monthly and stable releases
happened annually.

This was revolutionary. But process revolutions aren't sexy. Nobody writes news
articles "Perl implements sustainable release management." But we could finally
see a future where before there was uncertainty. That future, along with the
rest of what was going on in the mid-2000s FOSS scene—like Rails, Django, and
yes, even Perl 6—caused an explosion of innovation:

- Moose revolutionized object-oriented Perl (2006)
- Plack/PSGI modernized web development (2009)
- Mojolicious, Dancer, and Catalyst competed on web frameworks
- DBIx::Class became *the* ORM solution for Perl

I was organizing YAPCs during this period, and I watched amazing talks about
these innovations. The hallway track was electric with ideas. But we had a
problem that nobody was really seeing yet.

We really had no process for evolving the core language. The slogan at the
time: Perl is the Runtime, CPAN is the Language.

During that same period, Python added generators, yield from, async/await,
structural pattern matching, matrix multiplication, and type hints as well as
others. Most of these started out in some form or another on PyPI and migrated
their way into the core. You should [watch Chris Neugebauer's
talk](https://www.youtube.com/watch?v=GCgzTWrwzxs) about async/await from
KiwiPyCon 2022. They had social structures in place to move features from the
community to core in what appeared like a methodical, measured way.

We didn't.

## Disorganized Crime

Python has PEPs. JavaScript has TC39. Perl had... arguments on a mailing list.

Without a formal process for evolution, the only safe choice was no choice. We
evolved the 'feature' and 'experimental' pragmas, with all their social
contract baggage. Features might sit in limbo for years. Subroutine signatures?
Introduced in 5.20. Stable from 5.24. Not marked non-experimental until 5.36.
Seven years of 'can we use this in production?'

You can't blame Perl 6 for that. Yes, there was confusion regarding version
numbers. Java and JavaScript managed to figure themselves out. If we'd had the
ability to evolve the core language, we could have had a narrative to get past
the 5/6 problem. Moose came out in 2006, we didn't get the `class` feature
until 2023.

I remember conversations at conferences. Someone would demo something and say
"it's on CPAN!" and then someone would complain that it required installing
half of CPAN to run. That complaint probably cost us a couple dozen developers.

The process isn't done yet. Python has `import from __future__`—promises the
feature won't change. I only learned about
[`stable.pm`](https://metacpan.org/pod/stable) while writing this. It does the
same thing for Perl. It appeared in 2023. Not widely known.

## While We Were Looking Away, The Competition Moved In

In 1999, if you wanted to "get shit done" without C, Perl was the obvious
choice, but maybe also Java, TCL, or PHP. Today the market is lousy with
choices: Python, R, Go, Rust, TypeScript, Elixir, Ruby, Dart, Swift, Kotlin,
Scala, Julia, Crystal, Clojure, Zig, Nim, Haskell, F#, C#, Groovy, Lua, Gleam,
D, Erlang, Raku, OCaml, Carbon, Mojo, and more.

Here's one for you. I've got some code—Rust versus Perl, same problem. You tell
me which one is prettier:

```rust
use lazy_static::lazy_static;
use regex::Regex;
use std::collections::HashSet;

fn extract_hashtags(text: &str) -> HashSet<&str> {
    lazy_static! {
        static ref HASHTAG_REGEX : Regex = Regex::new(
                r"\#[a-zA-Z][0-9a-zA-Z_]*"
            ).unwrap();
    }
    HASHTAG_REGEX.find_iter(text).map(|mat| mat.as_str()).collect()
}

fn main() {
    let tweet = "Hey #world, I just got my new #dog, say hello to Till. #dog #forever #2 #_ ";
    let tags = extract_hashtags(tweet);
    assert!(tags.contains("#dog") && tags.contains("#forever") && tags.contains("#world"));
    assert_eq!(tags.len(), 3);
}
```

```perl
use 5.42.0;
use experimental 'keyword_all';

sub extract_hashtags ($str) {
    my @tags = $str =~ m/(\#[a-zA-Z][0-9a-zA-Z_]*)/g;
    return map { $_ => 1 } @tags;
}

my $tweet = "Hey #world, I just got my new #dog, say hello to Till. #dog #forever #2 #_ ";
my %tags = extract_hashtags($tweet);

die unless all { $_ } @tags{qw/#dog #forever #world/};
die unless (keys %tags) == 3;
```

Yeah, I thought so too.

Perl broke ground for a lot of things. CPAN was *revolutionary*—now it's table
stakes. Testing infrastructure with CI environments, built-in online
documentation systems—everyone has a solution now. And every language has an
angle. Go sells speed. Rust sells safety. Python? Python sells jobs. And Perl?
We sell backwards compatibility. Don't get me wrong that's a compelling
argument if you're in the know, but it's not gonna roll your socks up and down.

Perl doesn't have anything that the other languages don't, but it's also not
missing much that the other languages do—and what it's missing is still usually
found on CPAN.

## The Smoking Gun

Look, I love Perl. I have a White Camel Award sitting on my shelf. I've spent
years organizing conferences, building relationships, and advocating for this
language. But I can't pretend we didn't shoot ourselves in the foot.

Want to know what really killed Perl? It wasn't Perl 6 (though it had its
impact) and it wasn't TIMTOWDI (JavaScript has a *lot* of ways to do things).
Want to know what really killed Perl? Success. We were the child star who came
of age in the public eye, and had to learn to deal with the fact that the world
doesn't care. We had the features. We had the innovation happening on CPAN. We
had passionate, brilliant people building amazing things. But we spent crucial
years arguing on mailing lists and IRC. By the time we got our act together,
we'd lost a generation and that's on us.

## Colophon

The photo is "A Dark Mysterious Corridor with a Figure in Sight" by
[Victoria Wang](https://unsplash.com/@satou1983) on
[Unsplash](https://unsplash.com/photos/a-dark-mysterious-corridor-with-a-figure-in-sight-9D3PpbavMQ0).
