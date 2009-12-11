Title: RDF is *hard*  
Author: Chris Prather
Date: 2004-08-12 23:47:20

# RDF is *hard*
So I've been working on moving the Perl Monger XML over to XML/RDF. But like SemWeb Barbie says "RDF is *Hard*". I finally got a <a href="http://dahut.pm.org/pm_groups.rdf">small sample</a> to validate clean but I want to re-arrange things so that I can easily parse it using XML::LibXML or some other fairly simple parser, so it can then be transformed by Template Toolkit. 

Why I hear you asking? Well mostly so I (and others) can link to things like<a title="http://dahut.pm.org/dahut_group.faof" href="Dahut Foaf">this</a>. Dahut.pm will maintain it's membership rolls via FOAF. And Bender will (eventually) be able to parse that. So in theory he'll have a clue as to who his members are, and could answer questions about them.


