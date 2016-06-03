---
Title: Blawd: Now With Plack Control
Author: Chris Prather
Date: 2009-12-02
---

# Blawd: Now With Plack Control

miyagawa, the esteemed ring-leader behind the [PSGI/Plack](http://plackperl.org/) tonight in response to an idle comment about wanting to move Blawd's simple `server` command over to the PSGI standard supplied me with a quick patch to do just that. So as of the most recent commit `blawd server --repo myblog.git` should "just work" with any environment that speaks PSGI. The code itself was exceptionally elegant and I thought it deserved to be shown off.

	has host => ( isa => 'Str', is => 'ro', default => 'localhost' );
	has port => ( isa => 'Int', is => 'ro', default => 1978 );

	sub _build__http_engine {
    		my ($self) = @_;
    		HTTP::Engine->new(
        		interface => {
            			module          => 'PSGI',
            			request_handler => sub { $self->handle_request(@_) },
        		},
    		);
	}

	sub execute {
    		my $self = shift;
    		my $app = sub { $self->_http_engine->run(@_) };
    		Plack::Loader->auto( host => $self->host, port => $self->port )->run($app);
	}

The old version was based on [HTTP::Engine](http://search.cpan.org/dist/HTTP-Engine) the new version takes advantage of the HTTP::Engine plack support. 
