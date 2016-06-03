---
layout: post
Title: Something New  
Author: Chris Prather
Date: 2006-05-09 01:07:04
---

# Something New
I wrote a little program tonight that takes input from STDIN and posts to this blog using [Net::Blogger][1] (obviously if you're reading this it worked). Because it reads from STDIN it works nicely with [TextMate][2]. W00t. Here's the text of the script (posted using: cat `which blog` | mate | blog --title "Something New")
	#!/usr/bin/perl
	use strict;
	use warnings;
	use version; our $VERSION = qv("1.0.0");
	use Carp;
	use Config::Auto; 
	use Getopt::Long;
	use Net::Blogger

	$Config::Auto::Untaint = 1;
	my %conf = %{ Config::Auto::parse( '.blogrc', format => "yaml" ) };
	my %opt;
	GetOptions( 
		\%opt,
		'sitename=s',
		'title=s',
		'catagory=s',
		'proxy=s',
		'domain=s',
		'type=s',
		'username=s',
		'password=s',
	);
	$opt{sitename} ||= $conf{default};
	%opt = ( %{ $conf{ $opt{sitename} } }, %opt);
	{ local $/; $opt{postbody} = <> }

	my $mt = Net::Blogger->new(engine=>$opt{type});
	$mt->Proxy("http://$opt{proxy}");
	$mt->Username($opt{username});
	$mt->Password($opt{password});
	$mt->BlogId($opt{blogid});

	my $id = $mt->metaWeblog()
				->newPost(
					title=>$opt{title},
	                description=>$opt{description},
	                publish=>1
				) || croak $mt->LastError();

	__END__

[1]: http://search.cpan.org/~claco/Net-Blogger-1.01
[2]: http://macromates.com
