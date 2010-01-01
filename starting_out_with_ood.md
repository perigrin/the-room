Title: Starting out with Object Oriented Design
Author: Chris Prather
Date: 2009-12-31 19:21

# Starting out with Object Oriented Design

The [basic question][1][^1] for Object Oriented Design is 

>   For a given domain, how do you break the model up into Objects and
>   Classes?"

Let's say we're going to build an application to model the life cycle of a Book. The naive approach is to start asking "What different things are involved with Books, and how are they related?" The domain can be literally infinite when you start breaking it out and exploring it in detail.

This ability to break things up is part of the power of Object Oriented Design, because it seems like it should operate the way people have thought about the world for centuries. For the most part it does operate that way.  Without constraint, however, you end up lost trying to achieve some platonic ideal (like the questioner on Stack Overflow) or you end up with what people have come to call [Ravioli Code][2].

The platonic ideal trap is trying to figure out if a Book has an Author or a Author has a Book, and what to properly name the parent class of Books, Graphic Novels, and Magazines. It's involved with questions of should a Magazine have a "Table of Contents", Index, or Appendices? Should a Graphic Novel? Trying to sort these things out  is a fool's errand of fiddling details that may have no reflection on the needs of the system you're building. If the `cover_style` attribute is never called, does it matter that it might possibly return `'hardback'` for a Magazine?

The other end of the spectrum is Ravioli Code. Ravioli Code is what you get when you no-longer model discrete components in your logic, but rather model sub-components. It's like talking about a Book not as a collection of pages, words and phrases but rather as a sequence of glyphs and wood pulp recipes. The resolution of the focus of your model is too tight, not only can you not see the forrest for the trees, you can't see the tree for the wood grain.

The Power of object oriented modeling needs to be constrained by the needs of the application you're building. Say for example you're building a system for tracking a single book that is being written without considering it's publishing or distribution. In this system you won't need to model Sales, Stores, Advertising or any number of things dealing with the distribution chain, nor would you need to know about paper bond, binding type, covers, type faces, line height or the minutia of building the artifact that is a Book. You will however need to model things like Pages, Chapters, Paragraphs.

To bring this back around to the point, in any application of Object Oriented Design the goal is to have a Class or Object hierarchy that matches exactly as much of "reality" as you need to accurately describe and solve the problems you're writing a program to solve, and nothing more. There is a quote by Antoine de Saint-Exupery[^2]:

>   Perfection is achieved, not when there is nothing more to add, but
>   when there is nothing left to take away.

Instead of starting with "What different things are involved with Books, and how are they related?" we with really should start with "what features of a Book do I need to enable my application to deal with?"


[^1]: The accepted answer for this question on Stack Overflow is excellent.
[^2]: [Stevan][3] uses this quote all the time

[1]: http://stackoverflow.com/questions/1078838/oop-choosing-objects
[2]: http://en.wikipedia.org/wiki/Ravioli_code#Ravioli_code
[3]: http://stevan-little.blogspot.com/