Title: Baby Steps to NASCAR  
Author: Chris Prather
Date: 2009-03-05 00:30:32

# Baby Steps to NASCAR
chromatic has a <a href="http://www.modernperlbooks.com/mt/2009/03/the-relentless-progression-of-baby-steps.html">blog post</a> about taking things one small step at a time being easier than doing everything at once. He's right, if you want to run a marathon you need to learn to run a mile first.

That said, having worked for a company that was still shipping a Perl 4 binary, chromatic's analogy is broken. To model the thought process they had (have? I no longer work there) he's asking them to change their shoes every mile during their marathon.

This was a very large, reputable company. They had a very large very old code base. They had a continuous integration system, they had quarterly releases (or at least my group did), they could have had automated unit tests (but they didn't). They had a huge set of acceptance tests that were fairly easy to automate since the customers preferred we gave them consistent results rather than "better" results (unless of course we could prove exactly how this new result was better).

For them Perl was a tool, a set of shoes, rather than the race. They'd already gone through all the baby steps in their race, and for much of the code I worked with before CPAN existed. Upgrading Perl, or new modules from CPAN was just a new tool to get the same job done. A new set of shoes, or a dish washer that takes 30 minutes instead of 35 (or in the case of perl5.8.8, suddenly taking 35 minutes instead of 30 and we missed a deadline). 

I know he's not honestly saying that everyone everywhere should change the way they run, so that they can change their shoes every mile, but he seems to be looking at this from a Software is the Product perspective. For many companies, if they could generate their products using flatulent cows there would be some very highly paid cow fart collectors. Software most definitely is not the product, it's the shoe (or possibly the fart collector). I think a better metaphor here is racing, specifically NASCAR cause I live near Daytona and it's rubbed off on me a little.

You have to change your tires during a race if you want to win (or even finish respectably) the question is how often. There is a recommend frequency I'm sure, you should change your tires every N laps. Tire changes, like upgrades, take some time even if you get the process down to be finely tuned with experts at the switch. You also open yourself up to potential catastrophe, something could be wrong with the tire, or someone could trip up the switch over. Even if you have everything perfect and you've done it a hundred thousand times before there is always that chance that this time something is different and you'll end up with a blowout in the grass. So tire changes, and really all pit stops, become a tactical decision. 

How does this tactical decision affect Goodyear? It doesn't. They still produce new improved tires regularly. They still recommend how often you should change your tires. They also do everything they can to fail gracefully when people choose to ignore their recommended best practice. They were gonna do this regardless of the tactical decision because tire production isn't perfect and even though they have a thousand and one safety tests, the thousand and second condition will pop up and cause the tire to fail.

This is I think the lesson here, make regular steady progress, but fail gracefully. Deprecation cycles, and monthly releases are the way we make slow steady progress. Security patches, and [toolchain updates](http://use.perl.org/~acme/journal/38564) are ways that software can fail gracefully.
