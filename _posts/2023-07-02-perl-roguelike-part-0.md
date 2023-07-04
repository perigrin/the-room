---
layout: post
title: "A Roguelike in Perl Part 0 - The Setup"
author: "Chris Prather"
categories:
tags: perl, roguelike, dev, corinna
date: 2023-07-02
image: https://live.staticflickr.com/65535/51795828961_b3d3961af5_4k.jpg
---

## Introductions

Perl 5.38.0 just dropped, and with it comes (in my opinion) one of the most
exciting experiments in Perl, a new built-in object system. If `Moose` was the
postmodern object system for Perl then `feature 'class'` is post-postmodern,
it's a [meta-modern](https://en.wikipedia.org/wiki/Metamodernism) object
system.

The last couple of years I've not been paid to write Perl, so I've had to learn
a few other languages. I've discovered the best way for _me_ to learn a
language currently is to write a
[roguelike](https://en.wikipedia.org/wiki/Roguelike) game with that language.
There happens to be a pretty good [tutorial](https://rogueliketutorials.com/)
that has been copied to a bunch of languages, but not Perl.

So I thought let's kill two birds with one stone. Let's write a roguelike in
Perl using the new object system.

### Before We Begin

For this series of blog posts I'm going to assume you know Perl, but you
maybe haven't worked with it in a decade or two. I'm also going to assume you're
comfortable in a unix shell knowing how to run commands and edit files to
change those commands.

Maybe none of the above is true, you're still free to try to follow along. Just
don't freak out if things don't make sense.

### Getting Everything Installed

Since this is specifically about the new class feature in 5.38.0 you're going
to need a copy of that. My recommendation is to use
[plenv](https://github.com/tokuhirom/plenv) to set up a custom perl to work
with. Go follow Tokuhiro-san's instructions on getting plenv setup.

Once you have plenv set up, you'll need to create a directory to work in. On my
machine we'll go with `$HOME/dev/possessive_frogs` (I used github's [random repo name](https://alator21.github.io/repository-name-generator/)).
You can name yours whatever you'd like.

```
gh repo create --public --clone possessive_frogs
cd posessive_frogs
echo '5.38.0' > .perl-version
plenv install $(cat .perl-version)
plenv install-cpanm
```

Those last three lines create a `.perl-version` file to let plenv know which
version of Perl we're expecting to use, then tell plenv to install that
version of perl. We want the most recent version, at this writing 5.38.0.
Finally, we have plenv install `cpanm`, a simple CPAN client.

Now we need to install the game library. At this time, that library isn't
on CPAN, but it is on github:

```
cpanm https://github.com/perigrin/perl-games-rot.git
```

This will install the `Games::ROT` library and its dependencies. They're
minimal so it should "just work". That's the prep work done, let's write some code
and see if everything works!

### The Code

The new `class` feature only exists in perl with versions 5.38.0 or higher, so
let's start a new file `game.pl` and set a minimum version.

```
use 5.38.0;
```

Setting the minimum has a side effect of
enabling `strict` as well as a bunch of other features (in this case); `say`, `state`,
`unicode_strings`, `unicode_eval`,  `evalbytes`, `current_sub`, `fc`,
`postderef_qq`, `bitwise`, `isa`, `signatures`, and `module_true`. Check the
documentation for the [feature](https://perldoc.pl/5.38.0/feature) pragma for more on
what each of these do.

Next we'll need to enable the _experimental_ `class` feature. We explicitly
enable it with the `experimental` pragma.

```
use experimental  'class';
```

The `class` feature is experimental because it's still in development and may
change. The `experimental` pragma has been bundled with perl since 5.20.0
(2014) and it enables experimental features while disabling the warnings they
generate for being experimental.

We `use Games::ROT;`, bringing in the game utilities library. Then comes the new
stuff. Let's start by declaring a new Engine class to represent our game engine.

```
class Engine { ... }
```

The class keyword declares a new package which is intended to be a class. It
also enables all of the other new keywords within its scope. Keywords like the
`field` keyword, which introduces state variables into the class. Let's add a
`$height` and a `$width` field so our game has a definitive screen size.

```
class Engine {
    field $height :param;
    field $width :param;
}
```

These variables are lexically scoped and access the data in the underlying
object. With the new `class` feature, objects are no longer guaranteed to be any
specific kind of blessed data reference, they're intentionally opaque to help
promote encapsulation. It's best to just treat the data as a bundle of
lexically scoped variables.

This default toward encapsulation is different from existing Perl object
systems. Moose, Moo, and even the built in blessed data structure all end up
promoting very public APIs to access all of your state. The `class` feature
doesn't provide anything by default, certainly not in this early version. You
must explicitly declare which fields become parameters to the object
constructor, hence the `:param` attribute on the `$height` and `$width` fields.
This is why at the bottom we can pass `width => 80` and `height => 50` to
`Engine->new()`. If we didn't include them we would get an exception about an
required parameter missing. And if we include something that the class doesn't
know about, we get an error about unrecognized parameters.

```
 class Engine {
    field $height :param;
    field $width :param;

    field $app = Games::ROT->new(
        screen_width  => $width,
        screen_height => $height,
    );

    [...]
}
````

Let's leverage that encapsulation by having an `$app` field to hold our
`Games::ROT` utility. We can assign it right in the declaration and this
assignment is evaluated every time the constructor is run. We don't want users
of this class to be able to mess with the `Games::ROT` instance, so we don't
have a `:param` attribute on the field and we don't define any methods that let
us change it or even read it directly.

Having the `Games::ROT` object is great but it doesn't really get the game
started. `Games::ROT` has a `run` method that takes a callback for the game run
loop, so we could write something like this:

```
method run() { $app->run( sub { $self->render() } ) }
```

And then explicitly call `$engine->run()` later when we create an instance of
our engine class. But, I'd like the Engine to start running when it's created:

```
class Engine {

    [...]

    ADJUST {
        # start the app
        $app->run( sub { $self->render() } );
    }

    [...]
}
```

`ADJUST` blocks allow post-construction adjustments to the object. In this case,
we tell `Games::ROT` to run our main game loop: the `render` method.

```
method render() {
    my $x = $width / 2;
    my $y = $height / 2;

    $app->draw($x, $y, 'Hello World', '#fff', '#000');
}
```

For now let's just render something simple to the middle of the screen to make
sure that everything works.

Finally we need to create our game engine object and kick things off:

```
my $engine = Engine->new(
    width => 80,
    height => 50
);
```

All together it should look like this:

```
#!/usr/bin/env perl
use 5.38.0;
use experimental  'class';

use Games::ROT;

class Engine {
    field $height :param;
    field $width: param;
    field $app = Games::ROT->new(
        screen_width  => $width,
        screen_height => $height,
    );

    ADJUST {
        # start the app
        $app->run( sub { $self->render() } );
    }

    method render() {
        my $x = $width / 2;
        my $y = $height / 2;

        $app->draw($x, $y, 'Hello World', '#fff', '#000');
    }
}

my $engine = Engine->new(
    width => 80,
    height => 50
);
```

If we save our file and then run `carton exec perl game.pl`, we should get
`Hello World` printed nicely in roughly the middle of our terminal.

Tune in for Part 1, where we turn this hello world into something moving.

Thanks to [@tmtowtdi@mastodon.social](https://mastodon.social/@tmtowtdi/110651255031643872) for pointing out a capitalization bug in `render()`.
