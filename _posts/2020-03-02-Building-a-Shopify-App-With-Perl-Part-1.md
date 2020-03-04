---
layout: post
title: "Building a Shopify App with Perl (Part 1)"
author: "Chris Prather"
categories:
tags: shopify, perl
date:  Mon 02 Mar 2020 10:49:17 AM CST
image: https://live.staticflickr.com/8375/8376960724_e908c79b5c_z.jpg
---

## The Preface

My wife runs an afterschool pottery program in various elementary schools
around town. Parents sign their kids up for the classes, and then my wife's
staff go out to the various schools and teach basic ceramics to the kids. All
the existing student registration systems for this are both wildly expensive,
and they suck. She asked me if I could solve at least one of those problems
for her.

I thought maybe I'd write about the process.

## Part 0: The Rules

So I'm planning on building a product ready application that will (eventually)
have other developers involved, and possibly be open sourced. So let's lay out
some ground rules for what that actually means project wise:

1) Modern Perl 5 Best Practices

Perl has moved a long way since the dot-com era, and we should take advantage
of that. This means we'll be using perl-5.30, and make it as easy as possible
to test upgrading the version of perl to keep us within the [Perl Maintenance
Window][5]

2) Structured to encourage others to contribute

I know of at least one other person interested in this project for a
non-profit. I also know I work a full time job *other* than this project and
will eventually need help. So I want to ensure that the application is dead
simple for someone else to get up and get running at least in a dev enviornment
quickly so they can contribute as quickly as possible to the code.

This also means that there should be a robust test suite for everything.

3) Designed to minimize the differences between dev, staging, and production

I've been doing a lot of junior DevOps for the last couple years. I hate
differences beween my development environment and production and having to
account for those. So I want production and dev to be as close as possible.

## Part 0.5: A Modern Dev + Test Environment for People Who Overengineer DevOps

First thing we do is we go go github and create a new repository. I pick the
Perl 5 `.gitignore` file and since at the moment this isn't an open project no
license.

Then clone the new repository and start setting up the scaffolding. Because I
use [Carton][2], I start by dropping a cpan file of just the following:

```perl
on 'develop' => sub {
    requires 'Perl::Tidy';
};
```

This allows us to set a house style for the code layout. We may eventually
upgrade to soemthing like [Code::TidyAll][3] but right now we don't even really
have Perl yet.

Second thing we do is make sure that our dev, testing, and staging environments
are as similar as possible. I'm reminded recently of this meme:

<img src="https://pics.me.me/it-works-on-my-machine-then-well-ship-your-machine-62058450.png">

To make sure all my environments are a similar as possible, I built a
Dockerfile based off the [perl docker images][4] that basically installs some
basics (like carton, copies the working directory over,  runs `carton install
--deployment`, and starts the app up. Credit where it's due, this is based on
the docker files that sungo did for work.

```docker
FROM perl:5.30.

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y --no-install-recommends && \
    apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        carton \
        libssl-dev \
        libzip-dev \
        libpq-dev \
    && apt-get clean

RUN mkdir -p /usr/local/my-app
WORKDIR /usr/local/my-app

COPY . /usr/local/my-app
RUN rm -rf /usr/local/my-app/local

ARG VCS_REF="master"
ARG VERSION="v0.0.0-dirty"

ENV HARNESS_OPTIONS j6:c
RUN carton install --deployment

ENV LANG C.UTF-8
ENV EV_EXTRA_DEFS -DEV_NO_ATFORK

ENTRYPOINT ["carton", "exec"]
CMD ["morbo", "app.pl"]
```
I've also got a `docker-compose.yml` file that sets up a simple dev environment
using the image build from this Docker file.
```yaml
---
version: '3.3'
services:
registry:
    image: sacregistry:latest
    container_name: myapp-app
    restart: always
    environment:
        - SHOPIFY_API_KEY
        - SHOPIFY_API_SECRET
    volumes:
        - ./app.pl:/usr/local/sacregistry/app.pl
    ports:
        - 3000:3000
```
## Part 1: Start Living in the Past

Shopify has some lovely tutorials using various modern technologies like Node +
React, Node + Express, or even ancient legacy choices like Ruby + Sinatra. I
like Perl. The [steps for creating a Partner account][7] and an app are the
same. So just go follow them, I'll wait. You can go ahead and use [their setup
with ngrok][8] too, it'll play nice with our set up.

Now that you've done that let's replace their Step 3 with Perl. Becasue we want
others to participate in development we're going to be starting with
[Mojolicious::Lite][6]. Add the following to the `cpanfile`:

```perl
requires 'IO::Socket::SSL' => 2.009;
requires 'Mojoliicous';
requires 'Mojolicious::Plugin::Util::RandomString';
```
The extra requirement on `IO::Socket::SSL` is becasue without it the docker
image I'm using was defaulting to something lower and mojolicious was throwing
a fit. That may be different by the time you're reading this.


Next we build a simple app, create an `app.pl` and add the following:

```perl
    use 5.30.1;
    use Mojolicious::Lite -signatures;

    get "/" => sub ($c) {
        $c->render(text => "Hello, World");
    };

    app->run;
```
Then we can run `docker build -t myapp:latest .` to rebuild the app and
`docker-compse up` and it'll start up the app. This is great but we can't
really do much with this. So now we start laying in the shopify specific routes
(Part 4 in the the Shopify tutorial).

We should export the API key and API secret you got from Shopify into the
environment. In a bash shell I do the following:
```perl
export SHOPIFY_API_KEY="NOT A REAL KEY"
export SHOPIFY_API_SECRET="NOT A REAL SECRET EITHER"
```
These will get picked up by docker (sepcifically see the `environment` key in
the `docker-config.yml`) and passed through to the application.

And update our `app.pl` with a shopify route.
```perl
use 5.30.1;
use Mojolicious::Lite -signatures;

my $API_KEY    = $ENV{SHOPIFY_API_KEY};
my $API_SECRET = $ENV{SHOPIFY_API_SECRET};

plugin 'Util::RandomString';

get "/" => sub ($c) {
    $c->render(text => "Hello, World");
};

get "/shopify" => sub ($c) {

    my $shop = $c->param('shop') =~ s/\.myshopify\.com//r;

    my $scopes       = "write_orders,read_customers"; # TODO Replace with your own scopes
    my $redirect_uri = $c->url_for("/shopify/connect")->userinfo(undef)->to_abs;
    my $state = $c->random_string();

    my $url = "https://$shop.myshopify.com/admin/oauth/authorize"
    . "?client_id=$API_KEY"
    . "&scope=$scopes"
    . "&redirect_uri=$redirect_uri"
    . "&state=$state";

    $c->session(state => $state);
    $c->redirect_to($url);
};

app->run;
```

This will take an Invite link from Shopify (the tutorial explains how to
Generate those) and redirects to the [app authorization prompt][9]. The user
then verifies the permissions and can click an "Install" button which will send
them back to the callback route you whitelisted when you set up the app.

Next we'll need to handle that connect callback:

```perl
get "/shopify/connect" => sub ($c) {

    my $code = $c->param('code');
    my $shop = $c->param('shop');

    if ($c->param('state') != $c->session('state')) {
            die "Request origin cannot be verified"
    }

    if ( $shop !~ m|https?\://[a-zA-Z0-9][a-zA-Z0-9\-]*\.myshopify\.com/?| ) {
        $c->ua->post_p(
            "https://$shop/admin/oauth/access_token" => json => {
                client_id     => $API_KEY,
                client_secret => $API_SECRET,
                code          => $code,
            }
        )->then(
            sub ($tx) {
                my $access_token = $tx->result->json->{access_token};
                $c->render( text => "Got an access token, let's do something with it" );
                # TODO use the access token to access the store
            }
        );
    }
    else {
        die 'Bad Shop in Args';
    }
};
```

But as Shopify's tutorials point out we still need to [validate][10] that these
requests come from Shopify. So let's add the following helper to our `app.pl`:

```perl
use Digest::SHA qw(hmac_sha256_hex);

helper is_shopify_request => sub ($c) {
    my $params = $c->req->query_params->clone;
    my $hmac   = $params->param('hmac');
    $params = $params->remove('hmac');
    return 0 unless $hmac;
    return $hmac eq hmac_sha256_hex( $params->to_string, $API_SECRET );
};
```

now we can update the `/shopify/callback` handler with the following:

```perl
get "/connect" => sub ($c) {
    unless ( $c->is_shopify_request ) {
        die "HMAC validation failed";
    }
```

And it will validate the HMAC to ensure that the request came from Shopify and
nobody else.

Finally we should do something with the token we get back, not just be excited
we have it. So let's replace:

```perl
$c->render( text => "Got an access token, let's do something with it" );
# TODO use the access token to access the store
```

with something  like:

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

This fetches the `shop` endpoint json and displays it as plain text. The full
controller should look like:

```perl
get "/connect" => sub ($c) {
    unless ( $c->is_shopify_request ) {
        die "HMAC validation failed";
    }

    my $code = $c->param('code');
    my $shop = $c->param('shop');

    if ( $c->param('state') ne $c->session->{state} ) {
        die "Request origin cannot be verified";
    }

    if ( $shop !~ m|https?\://[a-zA-Z0-9][a-zA-Z0-9\-]*\.myshopify\.com/?| ) {
        $c->ua->post_p(
            "https://$shop/admin/oauth/access_token" => json => {
                client_id     => $API_KEY,
                client_secret => $API_SECRET,
                code          => $code,
            }
        )->then(
            sub ($tx) {
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
            }
        )->catch( sub { die "Coudn't fetch access token: @_" } );
    }
    else {
        die 'Bad Shop in Args';
    }
};
```
As the Shopify tutorial says in [Step 6][11] you should now be able to run your
application, with `docker-compose up` and hit it with the install link.`
---
The banner image is [The shop](https://flic.kr/p/dLf7z) by
[OiMax](https://www.flickr.com/photos/oimax), on Flickr.

This post has been edited based on feedback from people. If you want to see the
changes to this or any post on this site, it's all hosted [on github][12].

[1]: https://shopify.dev/tutorials/build-a-shopify-app-with-node-and-express
[2]: https://metacpan.org/pod/Carton
[3]: https://metacpan.org/pod/Code::TidyAll
[4]: https://registry.hub.docker.com/_/perl/
[5]: https://perldoc.pl/perlpolicy#MAINTENANCE-AND-SUPPORT
[6]: https://metacpan.org/pod/Mojolicious::Lite
[7]: https://shopify.dev/tutorials/build-a-shopify-app-with-node-and-express#step-2-create-and-configure-your-app-in-the-partner-dashboard
[8]: https://shopify.dev/tutorials/build-a-shopify-app-with-node-and-express#step-1-expose-your-local-development-environment-to-the-internet
[9]: https://shopify.dev/tutorials/authenticate-with-oauth#step-2-ask-for-permission
[10]: https://shopify.dev/tutorials/authenticate-with-oauth#verification
[11]: https://shopify.dev/tutorials/build-a-shopify-app-with-node-and-express#step-6-run-your-app[12]: https://github.com/perigrin/the-room/
