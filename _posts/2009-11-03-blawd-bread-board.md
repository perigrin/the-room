---
layout: post
Title: Bread::Board 
Author: Chris Prather
Date: 2009-11-03 10:30
---

# Bread::Board

So I've been working on [Blawd][1] some more this week, trying to clean up the code so I can get some of the TODOs out of the way. One of the big TODOs is to implement a configuration system.

I had built the application organically much the way I have built many different applications over time. I had a master `Blawd` class that wired everything Entries, Indexes, Storage and Rendering classes, together by passing around attributes to constructors. Pretty standard stuff really. The problem is that building a configuration out of this either you end up with a god object (one big object that gets passed around or is a Singleton), or lots of small attributes and methods for passing configuration around. None of these approaches appealed to me. Here's a pared down example from Blawd.

    package Blawd;
    use 5.10.0;
    use Moose 0.92;
    use namespace::autoclean;

    our $VERSION = '0.01';

    use Blawd::Storage::Git;
    use Blawd::Index;
    use MooseX::Types::Path::Class qw(Dir);

    has title => ( isa => 'Str', is => 'ro', required => 1, );

    has repo => (
        isa      => Dir,
        is       => 'ro',
        coerce   => 1,
        required => 1
    );

    has storage => (
        is         => 'ro',
        does       => 'Blawd::Storage::API',
        handles    => 'Blawd::Storage::API',
        lazy_build => 1,
    );

    sub _build_storage { Blawd::Storage::Git->new(gitdir => $_[0]->repo) }

    has indexes => (
        isa        => 'ArrayRef[Blawd::Index]',
        is         => 'ro',
        lazy_build => 1,
        traits     => ['Array'],
        handles    => {
            index      => [ 'get', '0' ],
            find_index => ['grep'],
        },
    );

    sub _build_indexes {
        my $self = shift;
        [
            Blawd::Index->new(
                title    => $self->title,
                filename => 'index',
                entries  => $self->entries
            ),
        ];
    }

    sub get_index {
        my ( $self, $name ) = @_;
        my ($entry) = $self->find_index( sub { $_->filename eq $name } );
        return $entry;
    }

    has entries => (
        isa        => 'ArrayRef[Blawd::Entry::MultiMarkdown]',
        is         => 'ro',
        lazy_build => 1,
        traits     => ['Array'],
        handles    => { find_entry => ['grep'], }
    );

    sub _build_entries {
        my ($self) = @_;
        [ sort { $b->date <=> $a->date }$self->find_entries() ];
    }

    sub get_entry {
        my ( $self, $name ) = @_;
        my ($entry) = $self->find_entry( sub { $_->filename eq $name } );
        return $entry;
    }

    __PACKAGE__->meta->make_immutable;

Enter [`Bread::Board`][2]. `Bread::Board` basically takes your app and wires it together for you. Rather than having dozens of replicated configuration attributes, you have a single block of code that wires up your application for you. This significantly streamlines the `Blawd` class because now we only need the things that `Blawd` is actually responsible for. Here's the new `Blawd.pm` …

    package Blawd;
    use 5.10.0;
    use Moose 0.92;
    use namespace::autoclean;
    use Blawd::Index;
    use Blawd::Entry::MultiMarkdown;

    our $VERSION = '0.01';

    has indexes => (
        isa     => 'ArrayRef[Blawd::Index]',
        is      => 'ro',
        traits  => ['Array'],
        handles => {
            index      => [ 'get', '0' ],
            find_index => ['grep'],
        },
        required => 1,
    );

    sub get_index {
        my ( $self, $name ) = @_;
        return unless $name;
        my ($idx) = $self->find_index( sub { $_->filename eq $name } );
        return $idx;
    }

    has entries => (
        isa     => 'ArrayRef[Blawd::Entry::MultiMarkdown]',
        traits  => ['Array'],
        handles => {
            find_entry => ['grep'],
            entries    => ['elements'],
        },
        required => 1,
    );

    sub get_entry {
        my ( $self, $name ) = @_;
        return unless $name;
        my ($entry) = $self->find_entry( sub { $_->filename eq $name } );
        return $entry;
    }

    __PACKAGE__->meta->make_immutable;
    1;
    __END__


First thing you notice is that this is 40% the size of the previous version, it's a full 32 lines shorter in our main application class and I didn't reduce the
`Bread::Board` version down the actual savings are larger. Second there are no builders, every attribute is `required`. This is because `Bread::Board` passes
everything into the constructor for us. This makes the code much more straight forward, this means that the homicidal maniac that picks up this code is six months
won't try to hurt me for *this* code.

But that extra code had to go somewhere. It went into a configuration routine for `Bread::Board`. The code for this lives in the [`Blawd::Cmd::Container` class][3], but here is a cleaned up example.

    my $c = container Blawd => as {

        service title => ( $cfg->title );

        service gitdir => ( $cfg->repo );

        service app => (
            class        => 'Blawd',
            lifecycle    => 'Singleton',
            dependencies => [ depends_on('indexes'), depends_on('entries'), ]
        );

        service storage => (
            class        => 'Blawd::Storage::Git',
            dependencies => [ depends_on('gitdir'), ]
        );

        service entries => (
            block => sub {
                my $store = $_[0]->param('storage');
                [ sort { $b->date <=> $a->date } $store->find_entries ];
            },
            dependencies => [ depends_on('storage'), ],
        );

        service indexes => (
            block => sub {
                return [
                    Blawd::Index->new(
                        filename => 'index',
                        title   => $_[0]->param('title'),
                        entries => $_[0]->param('entries')
                    ),
                ];
            },
            dependencies => [
                depends_on('title'), depends_on('entries'),
            ]
        );

    };

This code describes to `Bread::Board` that I have an 'app' which has 'indexes' and 'entries'. It also describes how to build a list of entries from a storage engine, and how to build a set of indexes from a list of entries. All of my logic on how to wire together the blog now lives in one place, this makes it much easier to build dynamically since this is the only piece of code that needs to know about a configuration object. This also means that if I later decide I want to switch out the storage I only have to change it in this one place rather than making sure that a long chain of accessors is wired together properly.

So the verdict? `Bread::Board` has made our code shorter and more readable, as well as increased flexibility. This is a win all the way around. I'm very happy I finally bit the bullet and tried it.

[1]: http://github.com/perigrin/blawd
[2]: http://search.cpan.org/dist/Bread-Board
[3]: http://github.com/perigrin/blawd/blob/master/lib/Blawd/Cmd/Container.pm
