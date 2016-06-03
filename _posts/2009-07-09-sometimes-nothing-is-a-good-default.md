---
layout: post
Title: Sometimes Nothing is a Good Default  
Author: Chris Prather
Date: 2009-07-09 12:34:20
---

# Sometimes Nothing is a Good Default
So I know this has come up before but I can't see where I've mentioned it and stevan just asked me to blog about it. One of the frequently requested features in Moose is for attributes to default to having some kind of accessor either `is => 'rw'` or `is => 'ro'` depending on the preference of whomever is making the request. This is because Moose defaults to *nothing* if you don't tell it what you want. I just ran into a situation where *either* default would lead to potential bugs in application.

Our application has a queue that when it hits a high-water-mark will trigger processing on the queue and then reset the queue. This isn't exactly a *common* design pattern but it should be at least a familiar one. Here is the attribute definition for the queue:

    has queue => (
        isa       => 'ArrayRef',
        lazy      => 1,
        default   => sub { [] },
        clearer   => 'clear_queue',
        metaclass => 'Collection::Array',
        provides  => {
            push  => 'enqueue',
            count => 'queue_size'
        }
    );

It's an `ArrayRef`, and is using the `Collection::Array` metaclass from [MooseX::AttributeHelpers][1] to provide a `enqueue` and a `queue_size` method that add elements and report the current size respectively. Notice that there is no `is => ` definition here, I'll explain why in a second. I also turned on the `clearer` option which gives us a `clear_queue` method that we will use in our flush code.

    sub flush_queue {
        my $self = shift;
        $self->do_something( $self->queue );
        $self->clear_queue;
    }

Next we have a `max_batch_size` that defines our high-water-mark. This is so we don't have to hard code a value but we default to 100. 
 
    has max_batch_size => (
        isa     => 'Int',
        is      => 'ro',
        default => 100,
    );

Finally we have our code that checks the queue size and flushes accordingly.

    after 'enqueue' => sub {
        my ($self) = shift;
        $self->flush_queue if $self->queue_size >= $self->max_batch_size;
    };

This uses an `after` method modifier that will trigger after each call to `enqueue`. Now since the queue flush event is *only* triggered after `enqueue` what would happen if we were to allow someone to replace the entire queue? We would have no (easy) way to check the queue size and flush appropriately potentially breaking whatever downstream code required us to batch in blocks of `max_batch_size`. We can still break the system here by providing a initialized queue to `new`, but we can easily fix that by adding an explicit `init_arg => undef` to our attribute like so:

    has queue => (
        isa       => 'ArrayRef',
        lazy      => 1,
        default   => sub { [] },
        init_arg  => undef,
        clearer   => 'clear_queue',
        metaclass => 'Collection::Array',
        provides  => {
            push  => 'enqueue',
            count => 'queue_size'
        }
    );

This makes for a robust queue implementation where the only (proper) way to modify the queue is through the `enqueue` and `flush` interfaces.

UPDATE: This is why it's good to have a proof reader, spot the bug in this article? I just did while writing unit tests for this code. I try to access the whole queue at one point:

           $self->do_something( $self->queue );

Which means that I need my attribute to have a read-only accessor. If I didn't do this then the logic of this article would stand and I wouldn't be sitting here with egg on my face. So remember kids, always ... erm never ... forget to have a proofreader.

[1]: http://search.cpan.org/dist/MooseX-AttributeHelpers
