---
layout: post
title: "Building a Shopify App with Perl & Javascript (Part 2)"
author: "Chris Prather"
categories:
tags: shopify, perl, javascript
date:  Mon 02 Mar 2020 10:49:17 AM CST
image: https://live.staticflickr.com/2914/33805528040_1cdb498599_5k.jpg
---

## The Preface

My wife runs an after-school pottery program in various elementary schools
around town. Parents sign their kids up for the classes, and then my wife's
staff go out to the various schools and teach basic ceramics to the kids. All
the existing student registration systems for this are both wildly expensive,
and they suck. She asked me if I could solve at least one of those problems
for her.

I thought maybe I'd write about the process. If you're discovering this for
the first time I refer you to [part 1][1] where I describe the goals and
the process in more detail, and talk about how I set up the integration with
Shopify.

[1]: https://chris.prather.org/Building-a-Shopify-App-With-Perl-Part-1.html

## A Quick Aside

My last post made it onto [r/programming][2].[^1] The only real question I got
was "Why" specifically "Why Perl". Since the answer I gave there sort of
impacts the decisions I'm about to talk about in this post I thought I'd
reiterate what I'd said.

My wife doesn't need a cobbler's children's shoes kind of scenario where I
spend all my time learning a new technology. So I'm limited to the languages I
know and work with regularly: Perl, JavaScript, and Go.

At the time I started this I didn't really know Go, and I hadn't ever written a
server application in it[^2]. So we're left with Node+Express or Perl.
Eventually we're gonna need to hire other people. Node is easier to hire for,
so it seems like Node would be the best fit. Except.

Except I've been a Perl guy for most of my career. I've spent a decade freelancing
Perl. I know where to find good Perl devs[^3], I know what a good Perl dev
looks like, and I can evaluate their output. I know how to set up a CI/CD
pipeline for Perl, and I know who's gonna be online at 3am when the Mojo::Pg
has started segfaulting and I need answers fast. I don't know that about Node.

This decision has absolutely nothing to do with the relative merits of each
language and their ecosystem. This decision is based around my needs and
experience. Like this decision should be for any company.

[2]: https://www.reddit.com/r/programming/comments/fi4r9l/building_a_shopify_app_with_perl_part_1/

## Hold your ears folks, It's showtime

After getting the basics of the backend application up and running. My thoughts
turned to the next most complicated piece: the frontend. It's been about two
years since I last did any kind of frontend webdev, I stopped because I knew it
was better for that project to hire someone than it was for me to learn the
tool-set. This project doesn't have that luxury.

So we need to remember how to piece together a modern(ish?) frontend dev
tool-set. Shopify highly recommends that we use React, sadly I don't know React.
I did pick up Mithril though. While I don't know enough [Mithril][3] to fix a
fundamentally flawed architecture. I'm confident enough I know enough to build
from scratch (and make my own mistakes).

We'd left off with the application able to be registered with Shopify stores
and doing the most trivial piece of work possible: authorizing a token and
getting shop details as JSON and then just rendering that as text. That is neat
but no where near what we need.

Also I had some feedback from [Joel Berger](https://twitter.com/joelaberger)
from the Mojolicious team. So where we ended last time with something like:

```perl
my $access_token = $tx->result->json->{access_token};
$c->ua->get_p(
    "https://$shop/admin/api/2020-01/shop.json" => {
        'X-Shopify-Access-Token' => $access_token
    }
)->then(
    sub ($tx) {
        $c->render( text => $tx->result->body );
    }
)->catch( sub { die "Request failed: @_" } );
```

It is now something like this:

```perl
my $access_token;
try {
	my $tx = await $c->ua->post_p(
		"https://$shop/admin/oauth/access_token" => json => {
			client_id     => $API_KEY,
			client_secret => $API_SECRET,
			code          => $code,
		}
	);
	$access_token = $tx->result->json->{access_token};
}
catch {
	die "Could not fetch access token: $@";
}

try {
	$c->stash( shop    => $shop );
	$c->stash( api_key => $API_KEY );
	$c->render( template => 'index' );
}
catch {
	die "Request failed: $@";
};
```

The line `$c->render(template => 'index');` tells the application to render the
contents of the template that is included inline at the bottom of the file in a
`__DATA__` section.

```perl
__DATA__

@@ index.html.ep
<html lang="en">
<head>
    <meta charset="utf-8">
</head>
<body>
 	<script src="https://unpkg.com/mithril/mithril.js"></script>
    <script>
   		m.render(document.body, m("h1", "Hello World!"))
    </script>
    </body>
</html>
```

This isn't exactly a tutorial on Mithril (yet), but a quick synopsis of what is
going on here. Mithril exports a single (global) function `m()` that is kinda a
jack of all trades. When called `m("h1", "Hello World")` it creates a DOM node
(think HTML tag but if you don't know what a DOM node is you probably should go
do some reading on modern web development), but because in Javascript
everything is an object the `m` also has it's own methods. The method
`m.render` takes a DOM node (`document.body`) and attaches the DOM it's
creating as children of that node. This means in the browser the above HTML
becomes something like:

```html
<html lang="en">
<head>
    <meta charset="utf-8">
</head>
<body>
 	<script src="https://unpkg.com/mithril/mithril.js"></script>
	<h1>Hello World</h1>
</body>
</html>
```

Instead of `m.render` Mithril also supports routing via `m.route`, it takes a
DOM node to render under, a default route, and then an object that defines the
*other* routes as keys with their vnodes (the return value of `m()` is called a
vnode) values.

Also, like React, Mithril also supports the concept of Components. A Component in
Mithril is just a JavaScript Object that has a `view` method. They can be
passed directly to `m()` and rendered. So we can do something like this:

```html
<head>
	<meta charset="utf-8">
</head>
<body>
	<script src="https://unpkg.com/mithril/mithril.js"></script>
	<script>

		let Nav = {
			view: () => {
				return m("nav.tabs", [
					m(m.route.Link, {href: "/dashboard"}, "Dashboard"),
					m(m.route.Link, {href: "/events"}, "Events"),
				])
			}
		};

		let Dash = { view: () => {return [m(Nav), m("section", m("h2", "Dashboard"))]} };
		let Events = { view: () => {return [m(Nav), m("section", m("h2", "Events"))]} };

		m.route(document.body, "/dashboard", {
			"/dashboard": Dash,
			"/events"   : Events,
		})
	</script>
	</body>
</html>
```

## I ain't never seen two pretty best friends

This works, click on the links and you can navigate between the components
rendered as different pages. It is however deeply ugly, so we bring in some
CSS. CSS has come a long way since I first learned it in the late 90s.
[Zed Shaw][4] has been talking about Flexbox and Grid layouts on
his twitter recently. So I took a stab at writing some.

```css
body {
    font-size: 16pt;
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: minmax(100%, auto);
    grid-template-areas:
        "nav cnt cnt cnt cnt cnt cnt cnt cnt cnt";
}

nav.tabs {
    grid-area: nav;
    background-color: grey;
    display: flex;
    flex-direction: column;
    border: 1px solid black;
    margin: 0 1rem;
}

nav.tabs a {
    color: black;
    text-decoration: none;
    border-bottom: 1px solid black;
    padding: 1rem .5rem;
    text-align: right;
}

nav.tabs a:hover {
    background-color: coral !important;
}
```

Stick this in a `<style></style>` block at the top of the HTML above and we
get:

![Screen Shot of the Above HTML][5]

That won't win prizes but it does look better. The trick to getting the layout
to work is the two new features [Flexbox][6] and [CSS Grids][7]. Flexbox is the
slightly older feature, but the advice that the Mozilla Developer Network gives
is that CSS Grids are for larger layouts and Flexbox is for smaller controls,
and that's how we use it. So let's look at CSS Grids first.

We start in the `body` definition:

```css
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: minmax(100%, auto);
    grid-template-areas:
        "nav cnt cnt cnt cnt cnt cnt cnt cnt cnt";
```

The line `display: grid;` enables the Grid layout inside the document body;
`grid-template-columns` says that we want 10 columns each 1/10th of the page (
`repeat(10, 1fr)` says give me 10 columns at 1 fraction (i.e. 1/10th) of page
each); `grid-template-rows` says give me a row that is 100% of height of the
page, or `auto` sized; finally `grid-template-areas` says "within the grid,
make areas with these names in this pattern", that is a single navigation
column, and then 9 content columns. Then in `nav.tabs` we can simply tell it we
belong in the `nav` area (`grid-area: nav`).

It's a lot, and it's complex, and really you should go read the Mozilla
documentation on it and play with it. It's super awesome compared to the old
ways of having to do all of this with CSS resets and grid classes.

To style the Navigation component we use Flexbox. Flexbox is not nearly as
complex as Grids but then what it brings to the table is much simpler; it
simply allows you to flexibly re-define the way sub-elements arrange
themselves, either in columns or rows. So in `nav.tabs` we have:

```
display: flex;
flex-direction: column;
```

The `display: flex` enables the flex box, and `flex-direction: column` says
we're doing a vertical orientation. That's it, everything else is classic CSS
for making the boxes be the right size and shape. I highly encourage you to
look at the tweets Zed made about this and to look deeper into the Mozilla docs on
both [Flexbox][6] and [Grids][7].

[3]: https://mithril.js.org
[4]: https://twitter.com/lzsthw/status/1343225343537242112
[5]: https://perigrin.keybase.pub/screenshots/screenshot-2020-12-30-154056.png
[6]: https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox
[7]: https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids

## Tying it all up with string

At this point things are getting kinda complex, the HTML for our template looks
like this:

```html
<head>
	<meta charset="utf-8">
    <style>
		body {
			font-size: 16pt;
			display: grid;
			grid-template-columns: repeat(10, 1fr);
			grid-template-rows: minmax(100%, auto);
			grid-template-areas:
				"nav cnt cnt cnt cnt cnt cnt cnt cnt cnt";
		}

		nav.tabs {
			grid-area: nav;
			background-color: grey;
			display: flex;
			flex-direction: column;
			border: 1px solid black;
			margin: 0 1rem;
		}

		nav.tabs a {
			color: black;
			text-decoration: none;
			border-bottom: 1px solid black;
			padding: 1rem .5rem;
			text-align: right;
		}

		nav.tabs a:hover {
			background-color: coral !important;
		}
    </style>
</head>
<body>
	<script src="https://unpkg.com/mithril/mithril.js"></script>
	<script>
		let Nav = {
			view: () => {
				return m("nav.tabs", [
					m(m.route.Link, {href: "/dashboard"}, "Dashboard"),
					m(m.route.Link, {href: "/events"}, "Events"),
				])
			}
		};

		let Dash = { view: () => {return [m(Nav), m("section", m("h2", "Dashboard"))]} };
		let Events = { view: () => {return [m(Nav), m("section", m("h2", "Events"))]} };

		m.route(document.body, "/dashboard", {
			"/dashboard": Dash,
			"/events"   : Events,
		})
	</script>
</body>
```

It's getting to be messy and distracting. And every time we make a change we
have to restart the docker container and reload the page, which slows down the
speed we can work at. Also keeping all the JavaScript in a single source file is
gonna get difficult to work with after a while. We need to break out the code
and provide a simple way to link it all back in.

Mithril's [Quick Start][8] has a section on getting started using [webpack][9].
Webpack is a bundler and preprocessor for JavaScript and other frontend tools. Mithil's
quick start section is a little out of date, but working with the Webpack [guide][10]
I managed to get something working for me.

Start by ensuring `npm` and `node` are installed, I used my system's package
management system for this. Then I created a new directory `assets/` in the
project repo. This way the JS world and the perl world are separated.

Inside the `assets/` directory to get things started with Webpack I ran:

```shell
npm init -y
npm install webpack webpack-cli --save-dev
```

The place where I think the Mitrhil stopped working is that Webpack 5 is more
biased toward configuration. I created the following configuration file[^5]:

```js
const path = require('path');

module.exports = {
    mode: "development",
    entry: './src/index.js',
    output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, "dist")
    },
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader']
            }
        ]
    }
}
```

First we establish that we're in development mode. Remember that we're in the
`assets/` directory, all the paths here will be relative to that.  Our entry
point, that is the main file for our application is located in `src/index.js`,
so I coped the Mitrhil code from my template above into that file. The output
file name should be `bundle.js` and it should live in the `dist/` directory.

Then as a bonus I have it take any file that matches `.css$` and add that file
to the bundle. It sets the content inline into the DOM when the code is loaded, so
I don't have to keep track of where my CSS is, Webpack just makes sure it's
there.

Finally I created a new index.html page in the `dist/` directory for
development purposes, at least until I need access to the Shopify environment.

```html
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Super Awesome Cool Registration UI</title>
</head>
<body>
    <script src="./bundle.js"></script>
</body>
```

Now I can start hacking out components and building the front end. During
coding sessions `npx webpack --watch` will track the `src/` directory and
re-compile the bundle as I make changes. Then I just need to reload the browser
and see if they're there.

[8]: http://mithril.js.org/installation.html#quick-start-with-webpack
[9]: https://webpack.js.org/
[10]: https://webpack.js.org/guides/getting-started/

## This is where you realize the app is a pile of hams in a trench coat

Obviously this isn't done yet, I've really just started the basic front end and
a lot of pieces are still falling into place. I haven't started any of the
database design work. In fact I woke up after only about 4 hours of sleep this
morning and couldn't fall back asleep because I was thinking about the various
entities in the design (Events have one or more sessions, Sessions are bundles
of Time + Location + Roster + Resources etc.) So obviously a bunch of things
may change.

When I last approached Mitrhil I was dealing with a large complicated
application where the components were having problems keeping updates happening
consistently. Digging around I'd stumbled across the [Meiosis Pattern][11]. I
spent some time trying to get back up to speed with it but I'm afraid it's
moved forward significantly in two years and I've forgotten whatever pieces I
needed to scale up from foxdonut's documentation. At some point if I decide to
refactor this application into using that I hope to produce a single good
tutorial about how to build a real world application using it[^4].

I also fully didn't expect it to take over 9 months to get back to working on
this app. Hopefully I don't end up with Part 3 sometime next September.

[11]: https://meiosis.js.org

---

The title image is "rustic yet modern" by [Christian Collins](https://www.flickr.com/photos/collins_family/) on Flickr.


[^1]: I'm as flummoxed as anybody.

[^2]: I have now, I've even ported most of stevan's [Web::Machine](https://metacpan.org/pod/Web::Machine) over to my own [webframework](https://github.com/tamarou/blackarachnia).

[^3]: I know that Leila at All Around the World and Mark at Shadowcat will return my calls in an emergency and give me a fair rate. I know that they'll recommend individual freelancers for me if I can't afford their costs.

[^4]: He has several example examples, some exceptionally advanced. They just weren't in a state where I, having forgotten half the JavaScript I knew two years ago, could piece apart the bits I needed to know.

[^5]: This didn't spring fully formed from my head like Athena, there was some back and forth arguing with webpack going on.
