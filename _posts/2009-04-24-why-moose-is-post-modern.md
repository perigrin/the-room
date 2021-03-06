---
layout: post
Title: Why Moose is Post Modern  
Author: Chris Prather
Date: 2009-04-24 16:57:20
---

# Why Moose is Post Modern
To steal from wikipedia:

> The term postmodern is described by Merriam-Webster as meaning either "of, relating to, or being an era 
> after a modern one" or "of, relating to, or being any of various movements in reaction to modernism that are 
> typically characterized by a return to traditional materials and forms (as in architecture) or by ironic 
> self-reference and absurdity (as in literature)", or finally "of, relating to, or being a theory that involves a 
> radical reappraisal of modern assumptions about culture, identity, history, or language"


Let's deconstruct this...

1)  "of, relating to, or being an era after a modern one" 

"Modern" object orientation in Perl came about with Perl5 roughly 15 years ago. It has been enhanced and extended somewhat since that time but that is the basis. Moose currently at age 3 post-dates that by 12 years. That's a generational gap, the people embracing Moose are the ones *after* the generation that modernized Perl to include Object Orientation. 

2) "of, relating to, or being any of various movements in reaction to modernism that are typically characterized by a return to traditional materials and forms (as in architecture) or by ironic self-reference and absurdity (as in literature)"

Moose is part of a movement in Perl that some would like to call the Enlightened Perl movement. This is an examination of Perl best practices learned over the 15 years of Perl5. Moose embraces the traditional materials and forms by encoding the generally agreed "best practices" for Perl object orientation (blessed hashes, accessor, no attempts at encapsulation enforcement by default ... ). Moose is also at it's heart self-referential in that it is built both upon Class::MOP which bootstraps itself, but also opens an API for extending itself via ... itself. As for absurdity, it's named Moose, so I really need to go further?

3) "of, relating to, or being a theory that involves a radical reappraisal of modern assumptions about culture, identity, history, or language"

Moose directly challenges the presupposition that Perl can't do proper Object Orientation. Moose not only provides an excellent base for generic object orientation (Classes, Attributes, Methods and standard introspection), but also provides a basis for applying more advanced forms previously only found in research projects (Traits/Roles, Advanced introspection and Meta programming, Alternative implementations). These features I think are "a radical reappraisal of modern assumptions". Culturally Moose unlike many FOSS projects is radically open with it's commit bits. You show up with a patch, you get a commit bit to apply that patch. As for history and language, I think my last point covered those succinctly.

Why is moose called "Post Modern"? First because Stevan thought it would be absurd to call a software project "Post Modern", because it is self-referential to a talk Larry gave about Perl being a post-modern language (and the evidence Larry gave for Perl there applies to Moose as well), and finally because by definition it is.


