Title: Command Line Moose or Knowing what You Don't know.  
Author: Chris Prather
Date: 2009-09-02 13:14:07

# Command Line Moose or Knowing what You Don't know.
In one of the comments on Dave Cross's hotly debated "Moose or No Moose" post someone said:

<blockquote>
To those who claim Moose is usable in command-line commands, please can you get Moose developers to stop saying otherwise, cos it's hard not to take their claims seriously.
</blockquote>

I'm a core Moose developer. I use Moose for command line apps, almost exclusively. [MooseX::Getopt](http://search.cpan.org/dist/MooseX-Getopt) is by far the easiest way I have ever found to write a command line app. [MooseX::AppCmd](http://search.cpan.org/dist/MooseX-AppCmd) and several other modules only make it better.

That said, I cannot in good faith *recommend* Moose for command line applications because there is an overhead where Moose will take by some estimates 0.2s (I haven't measured this recently) longer to start up than an equivalent non-Moose app. That 0.2s can be a show stopper for some  environments where lots of processes are started up very quickly. You get 1500 processes starting up 0.2s slower each that's a 5 minute delay and I have worked in an environment where that 5 minutes could cost you $100K USD.

The thing is we talk about if something is "recommended" or not in a vacuum, and it's not. Several members of the Moose core team use Moose for command line apps for one reason or another, we have since Moose 0.16 when the start up penalty was measured in seconds not tenths of seconds, and we're constantly pushing to improve the start up times. For example we helped out wherever possible with the recent sponsored work that has shaved over 20% off of Moose's start up costs. So at that level *obviously* we recommended Moose for command line apps. We recommend Moose for just about every damn thing possible because that's how we roll.

But at the same time our official policy has to take into account every possible environment out there, and every possible developer who comes looking to us for help with Moose and their project. We can't tell them Moose will instantly make things better when we don't know know if they have 1500 processes and a tight time constraint, or hard memory requirements, or a need to have a 100% pure perl tool chain, or a 0 dependency install base, or fellow developers who don't know or understand Object Orientation. Knowing all that we don't know all we can safely say is that if you are in a persistent environment, and if you don't have exceptionally tight memory requirements, and you can install XS modules, and you like Object Orientation ... Moose will probably make your life better.

I know that Moose has generally made my life better.
