# Two Perls

So I gave [a talk at TPRC this year](https://youtu.be/iAiJgwu5M_s) about the tools we'll need if we
want to keep writing Perl in a world full of language models, and one of the
threads I kept tugging on was types. The claim I didn't have enough stage time
for: we don't need to *add* a type system to Perl. It's had one since 1987 —
unenforced, and living in the *operators* rather than the *values*.

What I need to get across first is a distinction most of us never make,
because for thirty years there's been no reason to. There are two things in the
world called "Perl." One is a language. The other is a program. We use
the same word for both and almost always that's fine.

I'm going to write the *program* — the thing Larry
and the porters have been polishing since the Reagan administration — as `perl`,
lowercase, in code font. And I'm going to write the *language* as Perl, with a capital P. `perl` is a very good implementation of Perl. It is also the only one there has
ever been. I've come to believe that's a problem.

## Only perl can parse Perl

You've heard the line. *Only `perl` can parse Perl.* We say it a little
ruefully and a little proudly, and it's true, and there are real
reasons for it. But listen to what it's admitting: we have never
written the language down. There is no document I can hand you — or hand a
compiler, or a language server, or GPT — that says what Perl *is*. There's the
program, and the program's behavior, and a thirty-year tradition of running the
one to find out about the other.

Which was fine! For a long time the whole loop was: a human writes some Perl,
`perl` runs it, and if you want to know what a piece of code means you ask
`perl` by running it and seeing what happens. The implementation was the
specification. And since there was exactly one implementation, the spec was
never *ambiguous* — only unavailable, except by execution.

It's the arrangement Britain has with its constitution: there isn't one — not a
single written document you can pull off a shelf and read. What counts as
constitutional is scattered across old statutes, centuries of court rulings, and
unwritten convention, and what it *means* mostly gets settled when a
case comes up and a court rules on it. `perl` is Perl's court. There's no
founding text for the language, so what's genuinely Perl — as opposed to the
current interpreter's administrative habit — only gets decided when some program
forces the question and `perl` hands down a verdict — and codifies it. An unwritten
constitution is exactly as hard to reform, to hand to a second court, or to tell
settled principle apart from accident, as you'd guess. Writing it down isn't
tidying. It's a constitutional act.

The trouble starts the moment something that *isn't* `perl` needs to understand
your code. A refactoring tool. A syntax highlighter that wants to get the hard
cases right. A static analyzer. A language model
rewriting a function it's never seen before. None of them can lean on `perl` the way you do. They *could* run it and see — but
executing arbitrary code just to find out what it *means*, side effects and all,
is more trouble than it's worth, and tells you about one run anyway. So they need
a description of the language, and we never made one, so they guess.

And there are only two ways to guess. You can try to *reimplement `perl`*
— chase its behavior corner by corner, forever. Every attempt I know of inherits
`perl`'s choices wholesale — porting its C, or chasing its test suite until it
passes — or breaks compatibility so hard it becomes a *different* language, or was
abandoned years ago. Or you can use *heuristics* — pattern-match
your way to *probably right*, which is what the linters and the editors and
increasingly the LLMs do. Neither can ever be
finished, because the third option — "go read the spec" — doesn't exist.

I think this quietly holds Perl back. Every tool
we might want is either prohibitively expensive to build or doomed to be
approximate. We let
the language evolve into a singular implementation, so there's nothing to build
against except the interpreter itself.

The question I want to chew on here: if we didn't have `perl` but we wanted to
capture as many of Perl's semantics as possible — what would that involve? What
if we could take advantage of 40 years of research?

## I want Perl to be good

Let me pick the least abstract version of that question: I want Perl to be fast.
Fast and Good aren't synonymous, but Fast helps a language get better.

I have to be careful, because it's easy to say something dumb here. As an
interpreter `perl` is not naive. It compiles your source into an optree and runs
optimization passes over it. What it can't do is get out of its own way. Every `+` in your program goes through `pp_add`, which takes two
SVs, works out at runtime what they are, adds them, and boxes the answer back
into another SV. And it *reaches* `pp_add` through the runloop: the dispatch loop
walking the optree one op at a time, calling each op's C function through a
pointer. That's why "fast Perl" has,
for its whole life, meant "Perl, except we rewrote the hot loop in C." You don't
make Perl fast. You leave Perl.

"Fast" isn't just about clock speed. Another thing `perl` can't easily do
is hand you a *single binary* — the Go trick, where you build once and
ship one file that just runs. The interpreter has to be present, dynamically loading your XS and dragging your
CPAN tree along at startup. Folding all of that into one self-contained
executable can be notoriously miserable.
So "make Perl fast" bundles three wishes: stop boxing every value, stop
dispatching every op, and let me ship one thing.

Basically every serious optimizing compiler on the planet is built around Static
Single Assignment (SSA) form. And with it, the trick I want. Julia feels dynamic
to write, but its compiler infers concrete types and hands them to LLVM, and what
comes out the far end is native machine code running at highly optimized speeds.

## Katamari Damacy with your data

SSA is a way of writing a program down so that
every value is assigned exactly once and then never changes. It sounds like a
bookkeeping rule and it's secretly a superpower. Once every value is written
once and immutable, the optimizer can reason about it. It can prove
things. It can move work around, fold constants, keep a value in a register,
delete computations entirely, because it *knows* nothing will sneak in
and mutate the world behind its back. Optimizers love SSA because SSA has
already done the hard part — it's pinned every value to one identity.

If you've written any Rust, you've already lived in a version of this. A `let`
binding is immutable by default; when you want a "new" value you don't mutate the
old one, you *shadow* it — `let x = f(x)` — a fresh binding that leaves the old
value untouched behind it. Rust took the discipline SSA imposes *inside* a compiler and surfaced it
into the language.

So I go to put Perl into SSA — `perl` implements scalar values as something
called an SV. And what's an SV? It's Katamari Damacy with your data. If you never played it:
you roll a little sticky ball around a room and everything it bumps into sticks
to it and it grows and grows until you're shoving a cow and a bicycle and a small
building around the screen. That's the SV: a heap-allocated, reference-counted,
dynamically-retypeable box with room for an integer *and* a float *and* a string,
plus the capacity to carry magic, to be overloaded, to be tied, to turn into a
dualvar. Every value in your program rolls along accreting the machinery for
everything it might one day need to be.

The whole point of SSA is that values are immutable and assigned once. SV is
defined to be the ultimate mutable data structure. It's a total waste to use an
SV in an SSA form.

I want to be clear this isn't me dunking on the SV. The SV is *great*. `perl`
makes every value an SV and it works, and it's the right call for what `perl`
is.

## Types and coercions

SVs are about data mutating, SSA is about values transforming over time. How
could we describe Perl with transitions over type? Because of Moose, Types and
Coercions came to mind.

The answer is that Perl has a Type system we've never bothered to write down. If
you squint, SVs really implement Types and Coercions between those Types. Let me
show you — it's simpler than it sounds and it's hiding in plain sight.

## Is "42" a number?

Is `42` a number? Sure. Is `"hello"` a number? No. Is `"42"` — the *string* — a
number?

Yes? No? ...It depends.

That "it depends" isn't a cop-out. It's the whole idea. Whether `"42"` is
a number depends on what you're about to *do* with it and which sense of
"number" you mean. Two clean tests hide under that "it depends."

The first — the more intuitive — is **behavior**. Does the value actually
behave like a number? Real numbers do arithmetic that makes sense — add one and
it grows, subtract it from itself and you get zero. The string `"Inf"` is the
classic counter example: everyone thinks of it as just "a really big number,"
but ask it to behave and it won't — `Inf + 1` is still `Inf`, and `Inf - Inf`
isn't zero, it's `NaN`. Right shape, wrong behavior, so it isn't really a number.
That's the test your gut already runs — and it's murder to automate, because
"behaves correctly" means checking every operation the type has.

The second test is mechanical, and it's the one a compiler can actually run: a
**round trip**. Convert the value to the type you're asking about, convert it
back, and see if you got the same thing.

```perl
my $x    = "42";
my $n    = 0 + $x;    # adding 0 to string "numifies" it, coercing it to a number
my $back = "$n";      # "42"
$back eq $x;          # true — nothing lost, "42" really is a number
```

Try the same thing with `"hello"` and it falls apart:

```perl
my $x    = "hello";
my $n    = 0 + $x;    # 0, and a warning if you asked for one
my $back = "$n";      # "0"
$back eq $x;          # false — you just lost your value
```

That's the line between *being* a type and merely *coercing* to one, and it
matters. A hashref stringifies to something like `HASH(0x561f3a...)`, but you
can't turn that string back into the hash, so a hashref is emphatically not a
string. It coerces. It isn't. Round-trip is really a membership test — is this
value *really* a `Num`? — and since each type is just the subset of a broader one
whose values survive the trip, stacking those tests is what carves out the
subtype lattice. It isn't perfect, though: `"Inf"` sails right through it, so
behavior still decides the hard cases.

Two tests, then: does it behave, and does it survive the round trip? Run those
over all of Perl's values and a hierarchy just falls out — integers are a kind
of number, numbers are a kind of string, references are their own thing —
without anyone declaring a single type. That's what I mean by a *latent* type
system: the types are already in the system. And notice: we define a value by
the operators that can meaningfully apply to it, and we already know what types
those operators take — `+` takes numbers, `.` takes strings. So just by looking
at what operations are done, we can infer what those values should be, like Julia
does, without you annotating anything.

## The SV is a checklist

Turns out the ivory-tower type-theory stuff solves the grubby compiler problem.
If you unwind all the mutations made to an SV as a series of coercions between
immutable values, you get the same list of behaviors.

Read that list from the compiler's side and it's everything you have to rule out
to drop a value into a bare machine integer — no mutation, no aliasing, no magic,
no overload, no tie, no dualvar. Read it from the language's side and it's the
contracts that make a value *really* an integer, *really* a string, *really* a
dualvar. Same list, one for one. So
describing what a value *is* and earning the right to make it fast turn out to be
the same act; where the guards hold you drop the SV, where they don't you keep
it. A licensed bridge, not a magic wand.

## Not one thing, and not five

When I've heard or had this conversation in the past, the sharpest Perl people I
know immediately reach for structural types or the interpreter types (SV et al.)
… and they usually object to this more traditional type system.

Stop describing `perl` and start describing Perl and it all falls out.

**Values.** Static — one committed datum, the way SSA wants them. They have
types, and the types form a lattice: `Int <: Num <: Str <: Scalar`, `Scalar <:
List`, references off to the side.

**Coercions and dispatching**, driven by *type signals*. This is what people mean by context,
and it isn't a type — it's a signal of what type is expected next. `+` signals
"Num." `my @x = …` signals "List." A value answers two ways. It **coerces**: a
static value plus an expected type gives you what it should become, computed on
the spot and never stored (storing every answer at once is the SV again). Or the
*operation* **dispatches** on the signal — `reverse` runs a different computation
depending on what's asked:

```perl
my @r = reverse(1, 2, 3);   # (3, 2, 1)  — list expected
my $s = reverse "hello";    # "olleh"    — scalar expected
```

`wantarray` is a function reaching out to read the signal it was handed —
return-type polymorphism, the thing Haskell needs typeclasses for, sitting in
the grammar. Coercion and dispatch are siblings, a value answering "what's
wanted of me here." And the signal is right there in the source, which is why
none of it *has* to wait for runtime: a compiler reads the context at each call
site and settles it then and there. `perl` just chooses to ask at runtime
instead.

**Enforcement** — the one nobody lists, and the only one that matters. `perl`
checks whether a coercion *exists*: `splice $x` is rejected, because nothing
turns a scalar into an array. But let a coercion exist, however deranged, and
`perl` runs it without a word — `"hello" + 5` is `5`, and `[1,2,3] + 5` adds five
to a memory address. The machinery to say *no* is right there; it says no to
`splice $x` and yes to adding a string to a number. `perl` enforces that a
coercion is *possible* and never that it's *sane*. That isn't a missing type
system. It's a type system with the one useful check switched off.

One value breaks the "static" rule, and it's the exception that proves it. A
dualvar — `$!`, the error variable — carries a message *and* a number,
independently:

```perl
open my $fh, '<', '/nope' or do {
    warn "message: $!";        # No such file or directory
    warn "number:  ", 0 + $!;  # 2
};
```

That's a value pretending to be a box: SV-shaped, the one place `perl`'s
representation pokes up through the language. Every other value commits to a
single datum and lets coercion compute the rest. The dualvar refuses to commit —
which is exactly why it's cleanly neither a string nor a number, and why, to
explain it, even the best Perl programmers reach back for the SV.

## The list `perl` can't count to

Here's a smaller one that kept embarrassing me while I wrote this. Every time I
tried to pin down what a Perl *list* is, I'd reach for some obvious property —
and the property would belong to `perl`, not to Perl.

Are lists finite? Feels obvious. But nothing in the *language* says so. Write
`my @x = 1..9**9**9` and that's a perfectly legal Perl list; `perl` just tries
to build the whole thing at once and eats all your memory. That's not Perl
drawing a line — that's `perl` running out of RAM.

Fine, are they at least eager? Also no. Nothing in Perl says `1..9**9**9` *has*
to be built the instant you name it. `perl` happens to be eager; Raku is lazy; a
third implementation could be lazier still and only do the work when you ask for
something that genuinely needs the whole list. Watch:

```perl
my @x = 1 .. 9**9**9;
print $x[-1];
```

`perl` dies. Not because the answer is hard — the last element of a range is
just its endpoint, `9**9**9` — but because `perl` insisted on constructing all
of it first. An implementation that hadn't inherited that habit would just print
the number. Same Perl program, different `perl`, and one of them can't count to
the end of a range it wrote itself. It even fails at the *wrong moment*: eager
`perl` blows up at *construction*, when the only honest time to struggle with an
unbounded list is when you ask it something that truly needs the whole thing.

So finite, eager, when-the-side-effects-fire — none of it is written into Perl.
It's all `perl`.

## Throwing perl out of the room

Here's the move that ties it together, and it's why "make Perl fast" turns out
to be the same project as "understand Perl."

Point Perl at an SSA-based, LLVM-style target — one that genuinely cannot hold
an SV and cannot fall back on `perl`'s runtime for anything — and it can't cheat.
There's no interpreter in the room to ask. Either the intermediate
representation carries enough real information to lower each value to a machine
type, or it fails, loudly, right there. And that failure isn't a bug. It's a
map. Every place the target chokes is a place we hadn't finished saying what
Perl *is* — a spot where we were still quietly leaning on `perl` to mean
something on our behalf.

This is the difference between two projects that sound identical and aren't.
Reimplementing `perl` means chasing the interpreter's behavior forever — the
bottomless pit. *Re-deriving Perl* is the inductive opposite: throw the
interpreter out entirely, then add back only the things you're *forced* to add
to make real Perl programs work. Whatever you're forced to put back is Perl, the
language; whatever you never reach for was only ever `perl`, the program. This
is how you settle the question that has no answer while `perl` is the only
spec — whether a given behavior belongs to the language or the
interpreter. You don't settle it by thinking harder; you settle it by *building
the second implementation and watching what it can't do without.* A compiler
that can't link `libperl` is the sharpest instrument I've found for telling the
two apart, precisely because it refuses to let me confuse them.

And this is where the range `perl` couldn't count to comes back. Where `perl`
can actually run a program, it's your oracle — you match its *behavior*, mind,
not its SVs. But `1 .. 9**9**9` is a program `perl` can't run; it just dies. Out
past the edge of what the interpreter can build, the oracle goes quiet, and you
don't get to look the answer up — you get to *decide* it. Choosing that the
program prints its endpoint is the
exact moment re-deriving Perl turns into making Perl *better* than `perl`. The
interpreter's limits stop being the language's.

And here's the payoff I still find a little startling: "can lower this without a
runtime" and "can optimize this past `perl`'s per-op dispatch" are the *same
requirement*. The information a runtime-free target forces into the IR is exactly
the information an optimizer needs to go fast. So the description of Perl and the
road to a fast Perl are one artifact. You don't write down what Perl is and then,
as a separate step, make it fast. Writing it down *is* making it fast.

So — if we didn't have `perl`, what could we have? We could have an
implementation that knows what Perl *is*, independent of how any one program
happens to store it, and can therefore compile it to something that flies.
That's the answer. That's the thing I was waving at on the slide.

## The other Katamari

But speed was only ever the concrete case — the version of the argument sharp
enough to draw blood. It isn't the point.

Types make development nicer, and I don't need to sell anyone on that anymore;
Python grew them, TypeScript is nothing but, Ruby's sprouting them, Go and Rust
were born with them. Old news. The fresh part, for Perl specifically, is *why*
they'd show up: the work you do to make the language fast is the work of defining
what correct behavior even is. And once the compiler knows what correct behavior
is, it can tell the *developer* when they've wandered outside it — instead of
silently doing something, anything, and handing back a plausible-looking wrong
answer.

Because here's the other Katamari, the one aimed at us instead of the machine.
`perl`'s SV rolls up every possible future of a value for the runtime's sake. But
without a type system, *we* do the same thing: we roll up every value we run into
— a number, a string, an array in the wrong slot, a typo — and Perl cheerfully
sticks it to the ball and does *something* with it, whether or not that something
makes any sense. The same latent static type system that un-rolls the machine's
Katamari un-rolls ours. Speed and being-kind-to-the-developer fall out of the
very same artifact, because they're both just "the language finally knowing what
it means."

And underneath even that is the thing I care about most. As long as
"Perl" means only "whatever `perl` executes," we can't even *have the
conversation* about what Perl is. I watched it happen over and over while I was
working on this: every time somebody tried to talk about the language, the
implementation walked in and sat down — someone reached for an SV, and we were
back to talking about `perl` again. That's the foreclosure with teeth: it isn't
just that the conversation keeps getting derailed — it's that, with only one
implementation, the conversation has no ground under it. "Is that the language or
the interpreter?" isn't a hard question; it's an *unanswerable* one, until
there's a second Perl to decide it against. Speed was just the case concrete
enough that the interruption became impossible to ignore. The tool my talk was
reaching for isn't a faster interpreter and it isn't a smarter linter.
It's the ability to talk about the language at all, apart from its one program —
to teach it, to tool it, to reason about it, to reimplement it, to *argue* about
it on solid ground.

## The most pluralist language in the world

There's an irony at the bottom of all of this that I can't stop turning over.

Perl's motto — the thing Larry gave us, the whole spirit of the culture — is
**TIMTOWTDI**. There Is More Than One Way To Do It. It is the most pluralist
language ever made. It will happily give you seventeen ways to write the same
loop and refuse, on principle, to bless one of them as correct.

And it has allowed itself exactly *one* way to be *done*. One implementation. The
most pluralist language in the world has been, at the level that matters most, a
monoculture — not because anyone decided it should be, but because we let the
language quietly collapse into its program and stopped noticing there was a seam
there at all.

Reclaiming the plural at *that* level — more than one way to run Perl, more than
one way to reason about Perl, more than one way to *define* Perl — is the whole
project. It's Larry's own motto, finally turned around and pointed at `perl`
itself. Chalk, the compiler I've been building and will no doubt bore you with in
later posts, is my run at it: not to replace `perl` — I'm not that foolish, and I
don't want to — but to prove there *can* be a second Perl, and in the proving, to
finally write down what the first one has been all along.

That's the part I keep coming back to. Somewhere in trying to make it fast, I
stopped being able to tell whether I was building a compiler or just, at long
last, taking dictation.
