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

## I get knocked down, but I get up again. You're never gonna keep me down.

So far we can explore a dungeon, and we can kill monsters, but we can't pick up
cool stuff. So let's add some cool stuff to pick up. Specifically let's give
ourselves a way to recover from the damage we take with some healing potions.

Let's start with a small refactor of our Entity class. Way back in part ? we
added the `$abilties` field to track the abilities of our entities, but really
the only entities with abilities are Mobs. We should move that field to the Mob
class.

```
t a/lib/Entities.pm b/lib/Entities.pm
index 30b4a3c..cbc9163 100644
--- a/lib/Entities.pm
+++ b/lib/Entities.pm
@@ -12,7 +12,6 @@ class Entity {
     field $bg : param              //= Colors::DefaultEntityBG;
          field $name : param            //= "<unnamed>";
               field $blocks_movement : param //= 1;
               -    field $abilities : param;
                   method x               { $x }
                   method y               { $y }
              @@ -20,7 +19,6 @@ class Entity {
                   method fg              { $fg }
                   method bg              { $bg }
                   method blocks_movement { $blocks_movement }
              -    method stats           { $abilities }
                   method name            { $name }
               method position() { [ $x, $y ] }
          @@ -31,11 +29,16 @@ class Entity {
               }
           }

       class Mob : isa(Entity) {
           use List::Util qw( first min );
           use Games::ROT::AStar;
           use Actions;
  +
  +    field $abilities : param;
  +
  +    method stats { $abilities }
  +
       # Mob's currently just walk toward the player if visible
       method next_action ($map) {
           state $fov = Games::ROT::FOV->new();
```
