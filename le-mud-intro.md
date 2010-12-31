Title: Party like it's 1979
Author: Chris Prather
Date: 2010-12-31

# Party like it's 1979

So recently I've been playing [Lacuna Expanse](http://lacunaexpanse.com) more
than I probably should be. It's an interesting game with a lot of other Perl
programmers involved, so the community feeling is very familiar. 

One of the things that Lacuna Expanse does that I haven't seen in other games,
not that I game a lot[^1], is they expose their server API fully. This means
not only can you use the official JS/HTML Client, or the iPhone client, or the
Adobe AIR client, but you can write your own scripts to target the API.
Shortly after the game was formed, `tsee` created
[Games::Lacuna::Client](http://github.com/tsee/Games-Lacuna-Client).

After that there was an explosion of small Perl scripts doing everything from
building reports of ships in the empire, to doing full automation of your
account through a limited AI "Governor".

Earlier this week in the in-game chat someone suggested there should be a
text-based version of the game. I suspect that the suggestion was in jest, a
text-based client to a Web Based game that has a GUI already is ... well
awesome but silly. It did however get me thinking. With Games::Lacuna::Client
it shouldn't be *that* hard to write. Two days later... 

Well here is a Screen Shot:

<a href="https://skitch.com/perigrin/r8ft8/173x36"><img src="https://img.skitch.com/20101230-q9qftqfyy9hnegdp6g59tse9cr.medium.jpg" alt="Screen Shot" /></a>
## Games::Lacuna::MUD

The code is still in developer release mode. It may never get beyond that
point. If you'd like to get started with Games::Lacuna::MUD simply do the
following.

On a box with Perl 5.12.2[^2] installed download a copy of this Games::Lacuna::MUD
either by cloning it or downloading a
[tarball](https://github.com/Tamarou/games-lacuna-mud/tarball/master).

I'm gonna assume you have CPANMINUS installed. If not it use your own
preferred client or install cpanm from the web.

    > curl -L http://cpanmin.us | perl - --self-upgrade 

Make sure you have Dist::Zilla installed, this will make installing the dependencies easier.

    > cpanm Dist::Zill

Once Dist::Zilla is installed simply do the following to install the dependencies required for Games::Lacuna::MUD

    > dzil listdeps | cpanm

Once the dependencies are installed you will need to have a developer API
Key that you can acquire from 
[this site](https://us1.lacunaexpanse.com/apikey).

With the developer key you can create a configuration file for Games::Lacuna::Client. 

    ---
    api_key: [KEY]
    empire_name: [EMPIRE]
    empire_password: "[PASSWORD]"
    server_uri: https://us1.lacunaexpanse.com/

Save this as `$HOME/.le-mudrc` and you can start the MUD client with the following

    > perl bin/le-mud.pl

Some basic MUD commands work: look, go planet, go building, leave building, quit

Beyond this the commands are still rudimentary. There is no help system
because I wasn't happy with the on that I had and haven't come up with a slick
way of integrating a cross cutting concern like that. You are free of course
to view the source.

I'm really happy with the design of this project, I thought I'd take a few
blog posts to explain some of the tools I used to throw it together in about 2
days of work and how Moose and CPAN really helped make it work quickly and be
fun.


[^1]: Actually I tell people that my "game" is called Perl. When I want to do
something creative and non-work, I hack on throw away side projects. I've got
an addictive personality and I always feared I would waste far too much time
playing, which is why I've never started.

[^2]: I'm actually really lazy, `use 5.12.2;` is part of my stock boiler plate
for personal scripts though I don't think I use anything that wouldn't be
compatible under 5.10.