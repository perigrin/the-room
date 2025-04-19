# Code Reviews: Speaking to the Silent Audience

I've been thinking a lot lately about code reviews and pull requests, mainly because I've seen various debates about whether they're truly necessary or just bureaucratic overhead. The discussion usually focuses on efficiency: are formal code reviews worth the time they take away from "actual coding"?

This is a perspective that misses something fundamental about what code actually is.

## The Three Audiences of Code

Software has three distinct audiences, each with different needs:

1. **End Users**: People who use the software to get their jobs done
2. **Machines**: Compilers and CPUs that translate and execute our code
3. **Other Developers**: Engineers who need to understand, maintain, and modify the codebase

We spend a lot of time thinking about the first two audiences. User experience design centers on the first audience. Optimization, type systems, and compiler directives focus on the second.

But that third audience often gets the short end of the stick, especially when deadlines loom and feature pressure mounts. The argument goes: "If it works for users and runs on the machine, what does it matter how it's written?"

This perspective is how we end up with code that nobody wants to touch six months later—including the original author.

## Pull Requests as Performance

A pull request is a performance for that third audience. It's the moment when code transitions from "something I understand because I wrote it" to "something others need to understand without me explaining it."

When I submit a PR, I'm not just asking "does this work?" I'm implicitly asking:

- Can you understand what this code does without me explaining it?
- Does this make sense within our broader architecture?
- Would you be comfortable debugging this at 3am during an outage?
- Will this still make sense months from now when we've forgotten why we needed it?

That's a fundamentally different evaluation than "does it compile?" or "does it satisfy the requirements?"

## The Alternatives Argument

The common counterargument is that practices like pair programming and mob programming eliminate the need for formal code reviews. After all, if multiple developers collaborate on writing the code, haven't they already reviewed it?

In a limited sense, yes. But these approaches miss something crucial about the third audience problem.

When we pair program, we're still in the mindset of authors. We're explaining our thinking to each other, talking through decisions, and building shared context. This collaborative authorship doesn't simulate what happens when a completely uninvolved developer needs to understand the code months later.

Pull requests create a moment where the code must speak for itself—precisely the situation future maintainers will face.

## A Real-World Example

I had a stark example of this a few years ago. I inherited a codebase where the team had relied entirely on pair programming without formal reviews. The code quality was genuinely good. Individual functions were well-written, and test coverage was excellent.

But without the forcing function of having to explain the code to uninvolved reviewers, there were massive hidden assumptions throughout the system. The architecture made perfect sense to the tight-knit team that developed it together, sharing context through daily conversation. But for newcomers, it was nearly impenetrable.

Onboarding new developers was painful and slow. And when we later needed to modify assumptions that were baked into the core of the system, it was excruciating.

The problem wasn't bad code—it was insufficient empathy for that third audience.

## How We Might Do Better

I'm not suggesting that every team needs a heavyweight, bureaucratic process around code review. But I am suggesting that we need explicit mechanisms to ensure the third audience is considered.

Here's what seems to work best in my experience:

1. **Separation of Authorship and Review**: Have at least one person review the code who wasn't involved in writing it.

2. **Documentation Through Review**: The PR description and comments become documentation that explains not just what the code does, but why certain decisions were made.

3. **Context-Appropriate Process**: For trivial changes, lightweight review might be sufficient. For core architectural changes, deeper review is warranted.

4. **Complementary Practices**: Pair programming and code reviews aren't mutually exclusive—they address different needs and can work together.

## The Cost of Ignoring the Third Audience

The cost of ignoring the third audience is rarely visible immediately. Code that only makes sense to its authors doesn't cause test failures or user complaints right away.

But the cost compounds over time as the codebase evolves, original authors move on, and institutional memory fades. Eventually, the cost manifests as:

- Slow onboarding of new team members
- Reluctance to modify "scary" parts of the codebase
- Duplicate functionality because existing code isn't discoverable
- Bugs introduced by developers who misunderstood existing behavior

By the time these problems become acute, they're extremely expensive to fix.

## A Personal Approach

In my own work, I've found that imaging a specific person—someone unfamiliar with the problem I'm solving but who might need to maintain my code—helps me write more maintainable software.

I imagine explaining my code to that person without being present to add context. What would they need to know? What might confuse them? What assumptions am I making that aren't explicit in the code?

This mental model helps me spot the hidden assumptions and implicit knowledge that make code impenetrable to newcomers.

## Conclusion

Code reviews aren't just about finding bugs or enforcing style guidelines. At their best, they're a forcing function that makes us consider the third audience of our code—the future developers who will need to understand, maintain, and modify what we write.

Whether you use pull requests, pair programming, or some other mechanism, what matters is creating explicit space to consider this audience. Because sooner or later, all code becomes legacy code. And the kindest thing we can do for our future colleagues (and ourselves) is to write code that speaks clearly to its silent audience.

What do you think? How does your team ensure code remains comprehensible to developers who weren't involved in writing it? I'd love to hear your experiences and approaches in the comments.