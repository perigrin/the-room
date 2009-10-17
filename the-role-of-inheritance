Title: The Role of Inheritance  
Author: Chris Prather
Date: 2009-07-19 23:55:45

# The Role of Inheritance
Ovid [linked][1] to [a blog post][2] about the evil's of inheritance. The post
makes an excellent case for the dangers of inheritance for behavioral
composition. However I knew he was running into a problem when his first
example *still* used inheritance to denote [Type polymorphism][3]. That's
ultimately my problem with his "solution" to inheritance.

His final example however does not use inheritance, translating it a bit to
something [vaguely Perl-like][4] you get.

    class Ball extends MovieClip {
        has yMotionStrategy => ( does => 'NumberSequenceStrategy', is => 'ro' );
        has alphaStrategy  => ( does => 'NumberSequenceStrategy', is => 'rw' );        
        
        method handleEnterFrame(Event $e) {
            if (defined $self->yMotionStrategy) {
                $self->y($self->yMotionStrategy->nextValue);
            }
            if (defined $self->alpha) {
                $self->alpha($self->alphaStrategy->nextValue);
            }
        }
    }
    
    role NumberSequenceStrategy { requires 'nextValue';  }
    
    class AbsSineStrategy does NumberSequenceStrategy {
        has _frame => ( is => 'rw', default => 0);
        has _from  => (is => 'ro', init_arg => 'from');
        has _to    => (is => 'ro', init_arg => 'to' );
        
        public function nextValue():Number {
            $self->_frame($self->_frame+1);
            $self->to + abs(sin($elf->_frame / 15)) * ($self->from - $self->to);
        }
    }

Now my first issue is, what happens if I need a ball that has an
`xMotionStrategy`? I have to patch the source code. That's great if I control
the source, but if I don't I'm gonna have some issues. One solution is to
bundle the entire behavior into the Strategy.

    class Ball extends MovieClip {
        has behavior => ( does => 'enterFrameStrategy', is => 'ro' );
            
        method handleEnterFrame(Event $e) {
            $self->behavior->handleEnterFrame($e)
        }
    }

The problem here is that I disconnect my implementation from the interface,
one in the class and one in the behavior, and the behavior code really
inherits all of the issues we had originally so we have taken a step backward.
Another solution is to extract this behavior out into a Role.

    class Ball extends MovieClip with qw(yMotionStrategy alphaStrategy) {
        method handleEnterFrame(Event $e) {
            if (defined $self->yMotionStrategy) {
                $self->y($self->yMotionStrategy->nextValue);
            }
            if (defined $self->alpha) {
                $self->alpha($self->alphaStrategy->nextValue);
            }
        }        
    }

This works fine actually. If I need I can extend `Ball` with a
`xMotionStrategy` and a method modifier.

    class xyBall extends Ball with (xMotionStrategy) {
        after handleEnterFrame {
            if (defined $self->xMotionStrategy) {
                $self->x($self->xMotionStrategy->nextValue); 
            }
        }
    }

Oops I'm using inheritance here to specialize an existing class. This is one
of those places where inheritance actually works like it's supposed to.
However because of the [MOP][5] I don't have to use inheritance.

    Ball->meta->apply_role(xMotionStrategy);
    Ball->meta->add_after_method_modifier('handleEnterFrame', sub { ... })

This is much less intuitive, but just as functional. So with the proper use of
Roles and meta programming we can do without inheritance entirely. So
inheritance is pointless then right? Well yes if your entire world is flat and
disjoint.

Object Oriented programming started out simulating discrete events. It's
primary purpose is to help model the problem domain in to discrete well
objects. Inheritance is useful because in *some* problem domains things really
are specialized versions of other things. One of my projects at work right now
we have several different types of Client for a Vendor API. I have several
different occasions where I want to refer an instance but I don't care *which*
Class it is as long it is a Client. I could model this like:

    class Client with MooseX::Traits { ... }
    my $soapClient = Class->new_with_traits(traits => 'Role::SOAP');
    my $restClient = Class->new_with_traits(traits => 'Role::REST');

Composing a role to each instance at runtime. This would work but it isn't as
intuitive as:

    class Client { ... }
    Class SOAPClient extends Client with Role::SOAP { ... }
    class RESTClient extends Client with Role::REST { ... }

This composes the Role against the Class once for each combination ahead of
time. This means that I can look through the class definitions and know the
types of Client's I'm expecting to encounter. The Class hierarchy reflects the
mental model I want to portray of the problem domain.

Roles allow you to make your inheritance as deep as you need to properly model
the domain. But no deeper.

[1]: http://use.perl.org/~Ovid/journal/39291
[2]: http://www.berniecode.com/writing/inheritance/
[3]: http://en.wikipedia.org/wiki/Type_polymorphism
[4]: http://search.cpan.org/dist/MooseX-Declare
[5]: http://search.cpan.org/dist/Class-MOP

