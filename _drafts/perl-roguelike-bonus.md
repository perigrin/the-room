---
layout: post
title: "A Roguelike in Perl Part 8 - Stuff to Carry"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date: 2024-05-05
image:
---

## Introduction

This post is part of a series of blog posts following the [roguelike tutorial](https://www.rogueliketutorials.com/)
to demonstrate the new `class` feature in Perl 5.38.0.

In this post we're gonna start where we left off in [part 7](https://chris.prather.org/perl-roguelike-part-7.html). If you don't
remember what the code looked like you can refresh yourself with the listing
[here](https://github.com/perigrin/posessive_frogs/tree/part-7).

## Running in any dungeon

So now we've got our game, and we're pretty happy with it. But how can we share
it with our friends? We could just zip up the code and send it to them, but
that's not very friendly. We could write a script to package it up, but that's
a lot of work. Instead we're going to use the `Perl::Dist::APPerl` module to
package up our game for us.

[TODO: write the rest of the post]
