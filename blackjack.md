Title: Blackjack in 10 Lines of Modern Perl
Author: Chris Prather
Date: 2011-12-31 18:00:00

# Blackjack in 10 Lines of Modern Perl.

There is this [thread][1] on Reddit (and [StackOverflow][3]) about what's the
coolest thing you can pull off in 10 lines of code. Obviously the first
question people have is how do you define a "line", cause unless you have
significant whitespace issues you can do a lot on a single line.

If you limit yourself to a line is equal to a single idea or step in an
application you restrict things a lot, but it's still amazing what you can
pull off. For example, here's a game of Blackjack in 10 lines of code:

    use 5.14.1; use IO::Prompt; use List::Util qw(shuffle);
    sub deal { state $shoe = [ shuffle map { my $c = $_; map {"$c$_"} qw(❤ ◆ ♣ ♠) } ( 2 .. 10, qw( J Q K A ) ) x 6 ]; push $_[0], shift $shoe for ( 1 .. $_[1] ); $_[0]; }
    sub value { my $v; for ( local @_ = @{ shift() } ) { s/[ ❤ ◆ ♣ ♠ ]//; s/[JQK]/10/; $v < 11 ? s/A/11/ : s/A/1/; $v += $_; } $v; }
    sub show { say sprintf "%s (%i)", "$_[0] @{$_[1]}", value( $_[1] ) }
    my ( $player, $dealer ) = map { deal( $_, 2 ) } ( [], [] );
    while ( prompt( "@$player\nHit? ", '-tyn1' ) ) { if ( value( deal( $player, 1 ) ) > 21 ) { show( "Busted!", $player ); exit; } }
    while ( say("Dealer @$dealer") && value($dealer) < 17 ) { show( "Dealer busted!", $dealer ) && exit if value( deal( $dealer, 1 ) ) > 21; }
    value($player) >= value($dealer) ? show( "Player wins", $player ) : show( "Dealer wins", $dealer );
    
Simple no? 

Okay so this wasn't the most legible code I've ever written, let's run
Perl::Tidy on it and step through what's going. The first thing you'll notice
is that tidied up the code is 3x longer, but that's still only 30 lines of code.

    use 5.14.1;
    use IO::Prompt;
    use List::Util qw(shuffle);

We start with importing the stuff we'll need, we're going to rely upon David
Golden's work to make ArrayRef's work properly with push/shift. This feature
was introduced in 5.14 we we'll make a hard dependency on that, and also take
advantage of the other modern features this will import (like `say` and
`state`).

    sub deal {
        state $shoe = [
            shuffle map { 
                my $v = $_;
                map {"$v$_"} qw(&#x2764; &#x25C6; &clubs; &spades;)
                } ( 2 .. 10, qw( J Q K A ) ) x 6
        ];
        push $_[0], splice($shoe,  0, $_[1]);
        $_[0];
    }

The second thought, or major chunk, is dealing cards. This little subroutine
is doing a lot in a small space so let's break it out a little.

A deck of cards can be though of as the cartesian product of the values of the
cards (2..10, J, Q, K, A) and the Suits (&#x2764; &#x25C6; &clubs; &spades;).
We use nested `map`s to build this cartesian product. We do it for six decks
of cards because that gives the house better odds.

The six decks of cards are shuffled and stored in an ArrayRef inside a lexical
variable. We use a state variable so that we don't build a new shoe every time
we ask for a new card to be dealt.

Finally we actually deal the cards. The hand is passed in as `$_[0]`, and the
number of cards to be dealt is passed in as `$_[1]`. We just splice off of the
shoe and into the hand.

Now that we have a way to deal hands we need a way to calculate the value of a
hand.

    sub value {
        my $v;
        for ( local @_ = @{ shift() } ) {
            s/[ &#x2764; &#x25C6; &clubs; &spades; ]//;
            s/[JQK]/10/;
            $v < 11 ? s/A/11/ : s/A/1/;
            $v += $_;
        }
        $v;
    }
    
so the hand is passed in via `$_[0]`, since we're going to munge the value we
need to take a local copy of the hand. The idiom `local @_ = @_;` is what gave
me the basic idea.

Once we iterate through each card in the hand we start by stripping off the
suit since in Blackjack they're all equal. Then if it's a face card (but not
an Ace) we replace it with it's value. Can you spot the bug in the next line?

Aces in Blackjack are a little special, they can be either 1 or 11 depending
which one is to the player's advantage. If the value of the hand is less than
11 we'll want the Ace to be worth 11, if the value of the hand is *more* than
11 we'll want it to be worth 1. In the real world the value of a hand is the
same irregardless of the order of the cards, `Q 2 A` and `A 2 Q` should both
be worth 13, not 23. I spent far too much time trying to figure out how to
make that work here, and couldn't come up with anything simple. Patches
welcome.

Finally we return the running tally.
    
    sub show { say sprintf "%s (%i)", "$_[0] @{$_[1]}", value( $_[1] ) }

We had a common idiom for displaying a hand with some extra information so we
make a simple subroutine here that takes a String and a Hand and prints them
plus the value of the hand to STDOUT.

These last four chunks have all been setup for the main game. The game we've
implemented is a little different from Casino blackjack. I don't play
blackjack at a casino and I'm not a stickler for the rules. I may go through
and clean up this up to be more accurate ... but right now "meh".

    my ( $player, $dealer ) = map { deal( $_, 2 ) } ( [], [] );

Deal two cards to the player and the dealer.

Now we let the player choose to Hit or Stay. This line is a little convoluted
so let's dig into it.

    while ( prompt( "@$player\nHit? ", '-tyn1' ) ) {
        if ( value( deal( $player, 1 ) ) > 21 ) {
            show( "Busted!", $player );
            exit;
        }
    }
    
First we prompt the user what if they want to Hit. If they choose to take
another card check if the value of that new hand is greater than 21. If it is
we tell the player they've busted and quit. Eventually the player will either
bust or quit taking cards, and it'll become the dealer's turn.
    
    while ( say("Dealer @$dealer") && value($dealer) < 17 ) {
        if ( value( deal( $dealer, 1 ) ) > 21 ) {
            show( "Dealer busted!", $dealer );
            exit;
        }
    }
    
According to [Wikipedia][2] Dealer's in Blackjack are constrained to always draw if the value of their
hand is less than 17. So we draw for the dealer and perform the same check to see if they bust. 

Finally if nobody has busted.
    
    value($player) >= value($dealer)
        ? show( "Player wins", $player )
        : show( "Dealer wins", $dealer );

We check to see if the player's hand beats the dealer's hand and show the winner.

All told the original draft of the code took me less than an hour to write,
and the rest of the evening to clean up into something I'd be willing to show
off. It's this kind of thing that keeps me fascinated by programming, and part
of what got me started with it all those years ago.

[1]: http://www.reddit.com/r/programming/comments/nw8ve/what_is_the_coolest_thing_you_can_do_in_10_lines/
[2]: http://en.wikipedia.org/wiki/Blackjack
[3]: http://stackoverflow.com/questions/811074/what-is-the-coolest-thing-you-can-do-in-10-lines-of-simple-code-help-me-inspir