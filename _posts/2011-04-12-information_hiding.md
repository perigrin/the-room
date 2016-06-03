---
layout: post
author: Chris Prather
date: 2011-04-12 14:08:02 -0400
title: Information Hiding or how I learned to hate state and love behavior...
---

The conversations about documentation of Object Oriented Perl brought up a classic disagreement about the nature of encapsulation. It doesn't help that the term 'encapsulation' is overloaded.

Encapsulation is the wrapping of an object's data, also know as it's state, with guard behavior. This is so the state cannot be easily changed without the object being complicit. Think of it as the warranty sticker on an object, while you can (especially in Perl - through various levels of heroic effort depending on the object data type) violate the encapsulation, you break any guarantees that the object will continue to function as advertised. Just as with a broken seal, or a water damage indicator if we are talking Apple products, you get no guarantees from the manufacturer.

For many people, especially Geeks who are more often prone to violating a warranty as soon as they get the wrappings open. This may sound like a pain and hassle. In many ways it is a hassle, intelligent people should be allowed to take risks and do what they want or need with property they own. Even if that property is a copy of a project released under an Open Source license. On the other hand, the warranty seal really is a savings because you no longer have to worry about your object working correctly. So long as your doing what it says on the box, or documentation, it will do the right thing. If it doesn't you can rightly complain to the manufacturer and expect some level of help.

What then is the other definition of encapsulation, Information Hiding?

It really is just another step further in this same direction. Information hiding says that not only should nobody have a warrant to poke into the private parts of the object( including it's state), nobody even be aware of (or care about) how the inside is implemented.

Let me use an example that came up that I think illustrates it well. An example came up that we use $foo++ to increment the value in $foo. This suggests we should be able to use $obj->foo++ to increment the value of the foo attribute in $obj. With proper overloading and use of lvalues (and possibly ignoring some of the lurking dragons in Perl's implementations of these) it is possible and will keep true to the *first* definition of encapsulation.

This sounds really reasonable at first, and I admit I thought for a bit "well why not?" But I realized soon enough that what this did is created more work for myself. It meant I and any users of my objects were forced to violate Larry's Law of Laziness.

But, you say, how does it make us less Lazy? It reuses an idiom we already know. The quick dismissal is that you can't handle multiple arguments, for example in a table with cells. This is rightly countered with $obj->get_cell(1,3)++. Really the problem is more subtle.

By making a distinction between state and behavior we are forced us to be aware that similar things are treated differently. Even if we have proper encapsulation and *all* state is guarded with some behavior, we're still forced to be aware that some thing's behave like values; they increment, decrement, and are assigned with operators; while others will behave like ... well behaviors, that is like methods.

This then is the problem that information hiding is protecting. It means we have a uniform API, and it doesn't matter how the underlying implementation is handled. It means that I am protected from deciding at some future point that a given value is a poor choice and I can re-factor it out. It means that I don't have to play "Language Designer" and worry about how the increment operator plays with pure methods in my objects (what would $obj->book_hotel($client, $hotel)++ mean?). It means in short that I can in fact trust that my warranty is what I claim on the box.

