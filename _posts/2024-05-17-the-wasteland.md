---
layout: post
title: "Surviving in the Wasteland with Raylib and a Platypus"
author: "Chris Prather"
tags: perl, roguelike, dev, corinna
date: 2024-05-17
image: https://images.unsplash.com/photo-1616294252621-1d7d366f9079?q=80&w=2532&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

## Where the world ends is where you must begin.

I've had [a class][class] accepted at the Perl and Raku Conference in Las Vegas this
year. I'm going to be talking about how to build video games in Perl with a
focus on Roguelikes and RPGs. While preparing for this class I started looking
into the previous libraries for Game Development in Perl. SDL hasn't been
touched in years, and the SDL2 bindings wouldn't install cleanly on my computer
and when I got them installed they would segfault immediately.

I started looking at other libraries and all of them had similar issues. I
managed to get [TCOD][tcod] installed by hand compiling the library but it was
a pain and I didn't want to inflict that on my students. It was however looking
like that was the only option.

[class]: https://tprc2024.sched.com/event/1d68A/class-going-rogue-with-object-oriented-perl
[tcod]: https://metacpan.org/dist/TCOD

I complained about the situation on the Fediverse and [Brian Wisti][brian]
metioned [Graphics::Raylib][raylib-xs]. I had never heard of it before but it
looked like it was exactly what I was looking for. I installed it and it worked
perfectly. I was able to get a window up and running in a few minutes. Just
kidding, it also failed to compile. But [Alien::Raylib][alien-raylib] installed
cleanly as long as I had the Raylib library installed on my system. This was a
start.

[brian]: https://hackers.town/@randomgeek
[raylib-xs]: https://metacpan.org/dist/Graphics-Raylib
[alien-raylib]: https://metacpan.org/dist/Alien-Raylib

## Go then, there are other worlds than these.

Because of all the work I had done trying to get SDL, SDL2::FFI, and TCOD
installed I had dug quite a bit into the [FFI::Platypus][ffi-platypus] and
[Alien::Build][alien-build] ecosystem. I had started down the road of trying to
build an Alien::TCOD but I had gotten stuck on the fact that [libtcod][libtcod]
wants to be installed via Vcpkg and I have zero experience with that, let alone
how to automate it in an Alien::Build.

[ffi-platypus]: https://metacpan.org/dist/FFI-Platypus
[alien-build]: https://metacpan.org/dist/Alien-Build
[libtcod]: https://github.com/libtcod/libtcod

Alien::Build is kinda awesome if you haven't looked at it, especially the
[alienfile][alienfile] DSL that Graham Ollis has created. It's a really nice
way to check to see if a third party library is installed on your system and if
not to install it. It can automatically provide information to FFI::Platypus
based modules about where to find the library and how to link against it.
Something that TCOD could really use.

[alienfile]: https://metacpan.org/pod/alienfile

Since Alien::Raylib was already working I decided to see if I could build a FFI
wrapper around it using FFI::Platypus I started by looking at the Raylib.h
header file, which _is_ the official documentation, and attaching a few
functions to it.

```perl
use 5.38.2;

use FFI::Platypus;
use FFI::CheckLib;

my $ffi = FFI::Platypus->new(
    api => 2,
    lib => find_lib_or_die( lib => 'raylib', alien => 'Alien::raylib' ),
);

$ffi->attach( BeginDrawing      => []                         => 'void' );
$ffi->attach( ClearBackground   => ['int']                    => 'void' );
$ffi->attach( EndDrawing        => []                         => 'void' );
$ffi->attach( InitWindow        => [ 'int', 'int', 'string' ] => 'void' );
$ffi->attach( CloseWindow       => []                         => 'void' );
$ffi->attach( SetTargetFPS      => ['int']                    => 'void' );
$ffi->attach( DrawFPS           => [qw(int int)]              => 'void' );
$ffi->attach( WindowShouldClose => []                         => 'bool' );

InitWindow( 800, 600, "Testing!" );
SetTargetFPS(60);
while ( !WindowShouldClose() ) {
    BeginDrawing();
    ClearBackground( 0x000000 );
    DrawFPS( 0, 0 );
    EndDrawing();
}
CloseWindow();
```

With a little bit of work I was able to get a window up and running and drawing
the FPS in the top left corner. I then tried drawing "Hello World" and that
took a little more work, I had to add a custom type for the Color struct.

```perl

package Color {
    use FFI::Platypus::Record;
    use overload
      '""'     => sub { shift->as_string },
      bool     => sub { 1 },
      fallback => 1;

    record_layout_1(
        $ffi,
        qw(
          char     r
          char     g
          char     b
          char     a
        )
    );

    sub as_string {
        my ($self) = @_;
        sprintf "(red:%02x green:%02x blue:%02x alpha:%02x)", $self->r,
          $self->g, $self->b, $self->a;
    }

}

$ffi->type( 'record(Color)' => 'Color' );
```
Then I updated the ClearBackground function to use the new Color type.

```perl
$ffi->attach( ClearBackground   => ['Color'] => 'void' );
```

and added a new function for DrawText.

```perl
$ffi->attach( DrawText => [ 'string', 'int', 'int', 'int', 'Color' ] => 'void' );
```

And then I was able to draw "Hello World" on the screen.

```perl
InitWindow( 800, 600, "Testing!" );
SetTargetFPS(60);
while ( !WindowShouldClose() ) {
    BeginDrawing();
    ClearBackground(Color->new(r => 0, g => 0, b => 0, 255));
    DrawFPS( 0, 0 );
    DrawText( "Hello World", 10, 10, 20, Color->new(r => 255, g => 255, b => 255, a => 255) );
    EndDrawing();
}
CloseWindow();
```

Everything worked well enough I decided to turn this into a library. I copied
the contents of my test file over to a new module and started adding more
functions. I decided to copy the API from `Graphics::Raylib` as much as
possible. So starting there I added functions as I needed. I ported across the
simpler examples from the `Graphics::Raylib` docs and then started working on
getting the module installable via a CPAN client.

It has been a while since I've written a new module, and where as before I
would have used [Dist::Zilla][dzil] and automated everything, I discovered
[App::ModuleBuildTiny][app-mbtiny] and decided to give it a go. I've been
looking at how to pare down my development tools and this seemed like a good
place to start. I'm glad I did, it was really easy to use and I was able to get
an installable module up and running in a few minutes.

[dzil]: https://metacpan.org/dist/Dist-Zilla
[app-mbtiny]: https://metacpan.org/dist/App-ModuleBuildTiny

A quick test with an fresh perl install, a little environment pokery, and
`cpanm -l local https://github.com/Tamarou/Raylib-FFI.git` just worked. I have
a platform we can build video games on.

So if you'd like to see more about how to build video games in Perl, come to my
class at [The Perl and Raku Conference in Las Vegas][tprc-2024] this year.

[tprc-2024]: https://tprc.us/tprc-2024-las/

## Credits

The Photo is [white concrete house near bare tree under blue sky during daytime][photo] by [Simon Hurry][simon] on [Unsplash][unsplash].

[photo]: https://unsplash.com/photos/white-concrete-house-near-bare-tree-under-blue-sky-during-daytime-Fv-Y6lnl6pA
[simon]: https://unsplash.com/@bullterriere
[unsplash]: https://unsplash.com
```
