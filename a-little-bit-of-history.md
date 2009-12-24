Title: A Little Bit of History
Author: Chris Prather
Date: 2009-12-24 01:38

# A Little Bit of History

I recently did some digging for what may be some future writings, as well as possibly something larger. Part of those diggings involved research into Object Oriented programming's history. I found this example on wikipedia of Simula 67, the first Object Oriented programming language.

    Begin
       Class Glyph;
          Virtual: Procedure print Is Procedure print;;
       Begin
       End;

       Glyph Class Char (c);
          Character c;
       Begin
          Procedure print;
            OutChar(c);
       End;

       Glyph Class Line (elements);
          Ref (Glyph) Array elements;
       Begin
          Procedure print;
          Begin
             Integer i;
             For i:= 1 Step 1 Until UpperBound (elements, 1) Do
                elements (i).print;
             OutImage;
          End;
       End;

       Ref (Glyph) rg;
       Ref (Glyph) Array rgs (1 : 4);

       ! Main program;
       rgs (1):- New Char ('A');
       rgs (2):- New Char ('b');
       rgs (3):- New Char ('b');
       rgs (4):- New Char ('a');
       rg:- New Line (rgs);
       rg.print;
    End;
    
Here is a translation of the Simula 67 Example using [MooseX::Declare][mxd]

    {
        use MooseX::Declare;
        
        class Glyph {
            sub print { confess “Virtual” }
        }
        
        class Char extends Glyph {
            has char => ( isa => ‘Charecter’, is => ‘ro’ );
            method print { CORE::print $self->char }
        }
        
        class Line extends Glyph {
            has elements => ( isa => ‘ArrayRef[Glyph]’, is => ‘ro’ );
            method print { $_->print for @{ $self->elements } }
        }
        
        # Main program
        my @rgs = map { Char->new(char=>$_) } qw(A b b a);
        my $rg = Line->new(elements => \@rgs);
        $rg->print;
    }

There really isn't a point to this post, but I thought it was neat to see how nicely clean Object Oriented programming translates from it's earliest days to something very recent.

[mxd]: http://search.cpan.org/dist/MooseX-Declare
