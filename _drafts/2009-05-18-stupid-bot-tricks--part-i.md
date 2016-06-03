---
Title: Stupid Bot Tricks: Part I  
Author: Chris Prather
Date: 2009-05-18 15:22:40
---

# Stupid Bot Tricks: Part I
I thought I'd take a stab at showing off a bot framework I'd written at the
hackathon during the first [Frozen Perl][1].

The framework is known as either `Adam` or `Moses` depending on who you're
talking to, and is available on [GitHub][2] `Adam` lays all the framework and
`Moses` brings in the declarative sugar and magic. So let's take a look at a
quick bot I wrote to query [GitHub's API][3] and display user information.

    package UniBlab;
    use 5.10.0;

We start of defining a package for our Bot. Adam expects your package name to
be the Bot name, though you can override this later. Also we require 5.10.0
because we're gonna use the new `given`/`when` syntax, and because working
in a modern Perl won't make us cry.

    use Moses;
    use namespace::autoclean;

`Moses` exports all the `Moose`, `POE`, and `POE::Component::IRC::Common`
sugar that we'll need later on. It also resets our base class to `Adam`, which
is the default bot. `namespace::autoclean` will remove our sugar when we're
done just as it does in any `Moose` examples you may have seen.

    use POE::Component::Github;

We're going to be accessing the GitHub API so we'll use Chris Williams's
[`POE::Component::GitHub`][4]. Chris is also one of the main developers for
[`POE::Component::IRC`][5] which is the basis for `Adam`.

Next we specify which IRC server to join, and what channels we want to join on
that network.

    server 'irc.perl.org';
    channels '#bots';

Now we start implementing the bot itself. At this point it looks a lot like
your standard [`MooseX::POE`][6] class which really shouldn't be surprising since
it is. We create an attribute to store our GitHub POE component.

    has github => (
        isa        => 'POE::Component::Github',
        is         => 'ro',
        lazy_build => 1,
    );

    sub _build_github { POE::Component::Github->spawn() }

We set up a method for searching for users, and an event to catch the return
value and do something with it.

    sub show_user {
        my ( $self, $name, $nick, $channel ) = @_;
        $self->debug("searching for $name");
        $self->github->yield(
            'user', 'show',
            {
                event    => '_show_user',
                user     => $name,
                _channel => $channel,
                _nick    => $nick
            }
        );
    }

    event _show_user => sub {
        my ( $self, $resp ) = @_[ OBJECT, ARG0 ];
        my $d = $resp->{data}{user};
        $self->privmsg( $resp->{_channel} =>
                "$resp->{_nick}: $d->{login} is $d->{name} ($d->{location}) $d->{email} $d->{blog}"
        );
    };

Finally we set up a hook to `POE::Component::IRC` to catch our commands. This
is where the `given`/`when` syntax really comes into play for our command
dispatcher.

    event irc_bot_addressed => sub {
        my ( $self, $nickstr, $channel, $msg ) = @_[ OBJECT, ARG0, ARG1, ARG2 ];
        my $nick = parse_user($nickstr);
        given ($msg) {
            when (/^show user (.*)/) {
                $self->show_user( $1, $nick, $channel );
            }
            default {
                $self->privmsg( $channel =>
                        "$nick: Sorry I'm not sure what to do with that" );
            }
        }
    };

Finally we set up a little call to the `run` method defined in `Adam` that
will start up our bot and connect it to IRC. 

    __PACKAGE__->run;
    1;
    __END__

There you go, under 60 lines and you have an IRC bot that will query GitHub
for user details. An example of it's output:

    19:32 <@perigrin> UniBlab: show user perigrin
    19:32 < UniBlab> perigrin: perigrin is Chris Prather (Oralndo) chris@prather.org http://chris.prather.org
    19:32 <@perigrin> UniBlab: show user bingos
    19:32 < UniBlab> perigrin: bingos is Chris Williams ()  http://use.perl.org/~bingos/journal/
    19:32 <@perigrin> UniBlab: show user nothingmuch
    19:32 < UniBlab> perigrin: nothingmuch is Yuval Kogman (The Intertubes) nothingmuch@woobling.org 
                     http://nothingmuch.woobling.org

I'll leave it as an exercise for the reader to expand this example to lookup
repository information. The code is available [on GitHub.][7]

[1]: http://www.frozen-perl.org/mpw2008/
[2]: http://github.com/perigrin/adam-bot-framework/tree/master
[3]: http://develop.github.com/
[4]: http://search.cpan.org/dist/POE-Component-Github/lib/POE/Component/Github.pm
[5]: http://search.cpan.org/dist/POE-Component-IRC/lib/POE/Component/IRC.pm
[6]: http://search.cpan.org/dist/MooseX-POE/lib/MooseX/POE.pm
[7]: http://github.com/perigrin/uniblab-bot/tree/master
