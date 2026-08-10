# Two Perls

So I gave [a talk at TPRC this year](https://youtu.be/iAiJgwu5M_s) about the tools we'll need if we
want to keep writing Perl in a world full of language models, and one of the
threads I kept tugging on was types. I made a claim in passing and then kept
moving, because I had a whole paper's worth of argument behind it
and a lot of slides still to go: that Perl has *already had* a static type
system since 1987, that we've never enforced it, and that it lives in the
*operators* rather than in the *values*. We don't need to *add* a type system to
Perl. We need to expose the one we've had the whole time. It came back around in
the Q&A — someone stopped me and asked, more or less, what I meant by
that. This is the long version of the answer I didn't have room for on stage.

What I need to get across first is a distinction most of us never make,
because for thirty years there's been no reason to. There are two things in the
world called "Perl." One is a language. The other is a program. We use
the same word for both and almost always that's fine.

I'm going to write the *program* — the thing you `apt install`, the C that Larry
and the porters have been polishing since the Reagan administration — as `perl`,
lowercase, in code font. And I'm going to write the *language* — the thing you
mean when you say "I wrote it in Perl" — as Perl, with a capital P. `perl` is an
implementation of Perl. It's a very good one. It is also the only one there has
ever been, and I've come to believe that's a bigger problem than it looks.

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

It's the arrangement Britain has with its constitution. There isn't one — not a
single written document you can pull off a shelf and read. What counts as
constitutional is scattered across old statutes, centuries of court rulings, and
unwritten convention, and what it *means* mostly gets settled when a
case comes up and a court rules on it. `perl` is Perl's court. There's no
founding text for the language, so what's genuinely Perl — as opposed to the
current interpreter's administrative habit — only gets decided when some program
forces the question and `perl` hands down a verdict. And an unwritten
constitution is exactly as hard to reform, to hand to a second court, or to tell
settled principle apart from accident, as you'd guess. Writing it down isn't
tidying. It's a constitutional act.

The trouble starts the moment something that *isn't* `perl` needs to understand
your code. A refactoring tool. A syntax highlighter that wants to get the hard
cases right. A static analyzer. A second implementation. A language model
rewriting a function it's never seen before. None of these have `perl`'s runtime
underneath them to fall back on. They can't just "run it and see." They need a
description of the language, and we never made one, so they guess.

And there are only two ways to guess. You can try to *reimplement `perl`*
— chase its behavior corner by corner, forever. This is the road PPI and the
`B::` modules and every brave soul who's ever written a Perl parser has walked,
and it never quite ends, because you're chasing a moving, undocumented target
whose only definition is itself. Or you can use *heuristics* — pattern-match
your way to *probably right*, which is what the linters and the editors and
increasingly the LLMs do. Both are guessing. Neither can ever be
finished, because the third option — "go read the spec" — doesn't exist. The
spec is the program.

That little circle is, I think, what quietly holds Perl back. Every tool
we might want is either impossibly expensive to build or doomed to be
approximate, and not because Perl is uniquely cursed. It's because we let
the language dissolve into its one implementation, so there's nothing to build
against except the interpreter itself.

Which is the question I want to chew on here. If we didn't have `perl`
— if we could set the interpreter down for a minute and stop letting it answer
for us — what kind of implementation *could* we have?

## I just want it to be fast

Let me pick the least abstract version of that question, because it's the one
that dragged me into this. I want Perl to be fast.

I have to be careful, because it's easy to say something dumb here. `perl` is
not a naive interpreter. It compiles your source into an optree and runs real
optimization passes over it. It is already an optimizing compiler.
What it *can't* do is get below its own machinery — and there's more of that than
just the boxes. Every `+` in your program goes through `pp_add`, which takes two
SVs, works out at runtime what they are, adds them, and boxes the answer back
into another SV. And it *reaches* `pp_add` through the runloop: the dispatch loop
that walks the optree one op at a time, calling each op's C function through a
pointer. Two taxes, paid on everything — the SV wraps every *value*, the runloop
dispatches every *operation*. That's the ceiling, and it's why "fast Perl" has,
for its whole life, meant "Perl, except we rewrote the hot loop in C." You don't
make Perl fast. You leave Perl.

And "fast" was never only about the clock. The other thing `perl` can't easily do
is hand you a *single static binary* — the Go trick, where you build once and
ship one file that just runs. `perl` is an interpreter that has to be *present*,
dynamically loading your XS and dragging your CPAN tree along at startup, and
folding all of that into one self-contained executable is famously miserable.
(Ask anyone who's tried to get a non-programmer to install something off CPAN.)
So "make Perl fast" quietly bundles three wishes: stop boxing every value, stop
dispatching every op, and just let me *ship the thing*. All three are facts about
`perl`. Not one of them is a fact about Perl.

So what's off the table isn't optimization in general. It's a
specific and specifically popular *kind* of optimization: the sort built on SSA
form, the intermediate representation basically every serious
optimizing compiler on the planet is built around. And with it, the trick I
want, the one Julia pulls off. Julia feels dynamic to write,
but its compiler infers concrete types and hands them to LLVM, and what comes
out the far end is native machine code running at C-ish speeds. That's not a
research fantasy. It's a shipping language a lot of people use to do exactly
that. A dynamic-feeling language *can* get native speed by figuring out its own
types and feeding them to a real backend.

Perl can't — not today. The runloop and the binary are real, but they're the
legible kind of problem; the *deep* one, the one that is the same
problem as understanding what Perl even is, is the SV. So that's the thread I'm
going to pull. Figuring out why is the whole game.

## Katamari Damacy with your data

SSA — Static Single Assignment — is a way of writing a program down so that
every value is assigned exactly once and then never changes. It sounds like a
bookkeeping rule and it's secretly a superpower. Once every value is written
once and immutable, the optimizer can reason about it. It can prove
things. It can move work around, fold constants, keep a value in a register,
delete computations entirely, because it *knows* nothing will sneak in
and mutate the world behind its back. Optimizers love SSA because SSA has
already done the hard part — it's pinned every value to one identity.

If you've written Rust, Erlang, or Elixir, you've already lived in a version of
this. Erlang is the purest: variables are *single assignment* — the SA in SSA,
made law. `X = 5` binds `X` once and for all; `X = 6` isn't a reassignment, it's
a failed match. Want a new value? New name — `X1 = X + 1` — which is exactly the
renaming an SSA pass does for you, except Erlang programmers have done it by hand
for thirty years. Elixir loosens the knot: names rebind, but the data underneath
is immutable and the old binding lives on in any closure that captured it. Rust
calls the same trick *shadowing* — `let x = f(x)`, a fresh binding rather than a
mutation — with real mutation the special case you ask for out loud, `mut`. Three
dialects of one idea: a value is assigned once and never changes, and a "new"
value is a new binding. That's SSA, surfaced into the language — which is why
these are the easiest places to feel what it is before we drag Perl into it.

So I go to put Perl into SSA, and I hit the question that
contains everything. I've got a node in my graph holding a value. What *type*
is it?

`perl` has an instant answer, and it's useless: it's an SV. And what's an SV?
It's a scalar in the only sense `perl` knows — a heap-allocated,
reference-counted, dynamically-retypeable box with room for an integer *and* a
float *and* a string, plus the capacity to carry magic, to be overloaded, to be
tied, to turn into a dualvar. An SV is a container built to hold any scalar Perl
will ever have and to *become* any other scalar at runtime.

It's Katamari Damacy with your data. If you never played it: you roll a little
sticky ball around a room and everything it bumps into sticks to it and it grows
and grows until you're shoving a cow and a bicycle and a small building around
the screen. That's the SV. Every value in your program rolls along accreting the
machinery for everything it might one day need to be.

And here's the rub — SSA has *already designed all of that away*. The whole
point of SSA is that values are immutable and assigned once. The mutation the SV
exists to support? Gone. The runtime retyping? Gone. So putting an SV on every
node means paying, on every single value, for a pile of capabilities the form
guarantees you'll never touch. The `42` in your loop counter is immutable and
used once. It does not need a heap box that *could* have become a tied,
overloaded dualvar. It needs to be `42`.

I want to be clear this isn't me dunking on the SV. The SV is *great*. `perl`
makes every value an SV and it works, and it's the right call for what `perl`
is. The SV is designed to be a mutable, ref-counted, dynamically-retypeable,
universal ball of magic — that's its job and it does it perfectly. Its job is
just the exact opposite of SSA's. It's built for a world of
mutation and aliasing and runtime retyping, and SSA is the world that deleted
all three. It's not wrong. It's overkill, on purpose.

## The bridge that shouldn't be there

Here's where I landed, and it's the sentence this whole post is really about:

> An SV carries the machinery for every possible future a scalar might have. SSA
> is a decision to only worry about *right now*. There should be no way to bridge
> them — but Perl has a latent static type system that gets us there.

Sit with the first two sentences, because they really are opposites. The SV is
about *potential* — it holds every future open, keeps every option alive,
refuses to commit. SSA is about *commitment* — this value, here, is
this, full stop, no becoming. One is a hedge against every possibility; the
other is a decision. You shouldn't be able to have both. How do you take a value
whose entire nature is that it *could be anything* and pin it to one
specific thing, right now?

The answer is that Perl — the language, not `perl` — has a type system we've
never bothered to write down, and it happens to be exactly the tool for
collapsing "every possible future" into "this, right now." Let me show you,
because it's simpler than it sounds and it's hiding in plain sight.

## Is "42" a number?

Start with the dumbest question I can think of. Is `42` a number? Sure. Is
`"hello"` a number? No. Now: is `"42"` — the *string* — a number?

Yes? No? ...Maybe.

That "maybe" isn't a cop-out. It's the whole idea in one word. Whether `"42"` is
a number depends on what you're about to *do* with it and which sense of
"number" you mean, and — this is the important part — it does not depend at all
on how `perl` happens to be storing it right now. Two clean
tests hide under the maybe.

The first is a **round trip**. Take the value, convert it to the type you're
asking about, convert it back, and see if you got the same thing.

```perl
my $x    = "42";
my $n    = 0 + $x;    # 42
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
string. It coerces. It isn't.

The second test is about **behavior**, because surviving the round trip isn't
quite enough. The string `"NaN"` round-trips through numeric conversion just
fine — out and back, no loss. But it doesn't *behave* like a number: `NaN`
isn't equal to itself, `NaN - NaN` isn't zero. Right shape, wrong behavior, so
it isn't really a number either.

Two tests, then: does it survive the round trip, and does it behave? Run those
over all of Perl's values and a hierarchy just falls out — integers are a kind
of number, numbers are a kind of string, references are their own thing —
without anyone declaring a single type. That's what I mean by a *latent* type
system: the types are already *in* the values whether you wrote them down or
not. And it's a *static* one in the sense that matters here: a compiler can
recover these types by inference, the way Julia does, without you annotating
anything. Latent, because it was always there; static, because a machine can
find it before the program runs.

## The SV is a checklist

Here's the part that made me put my coffee down, because it's where the airy
type-theory stuff and the grubby compiler problem turn out to be the same thing
wearing two hats.

Go back to the SV — the ball of magic, the machinery for every possible future.
Write out what that machinery is actually *for*. It's mutable, so a value might
change. It's ref-counted, so it might be shared. It's dynamically retypeable, so
it might turn into something else. It carries magic and overloading and the
dualvar trick, so an innocent-looking `+` might secretly be running somebody's
code.

Now write out what you have to *prove* in order to drop a value out of its SV and
into a bare machine integer: that nobody's going to mutate it, that its lifetime
is a storage detail and not part of the value, that its type is genuinely fixed,
that there's no magic or overload or tie lurking.

Those are the same list. Read it one way and the SV is a checklist of everything
that could go wrong with a Perl value. Read it the other way — from the
language's side — and the type system's behavioral rules are that same checklist:
`NaN` isn't a number, a dualvar isn't an integer, tie and overload break the
operation's contract. The conditions that make a value "really an integer" are,
one for one, the conditions that make it safe to compile as one.

Which means writing down what the value *is* and earning the right to make it
fast aren't two jobs. They're one. The description of the language and the
license to optimize it are the same artifact. That's the bridge that shouldn't
exist: the latent static type system spans it value by value, exactly where it
can prove the SV's open futures don't happen here. And where it *can't*
prove that — real magic, an honest dualvar, an actual tie — you keep the SV, and
you should. It's a licensed bridge, not a magic wand, which is why I
trust it.

## But Perl's types aren't one thing

When I started showing this idea around, something happened that I have to tell
you about, because it's the most Perl thing imaginable. Some of the sharpest
Perl people I know — people who understand `perl`'s guts far better than I do —
heard "Perl's type system" and immediately reached for the SV. To explain what a
dualvar *is*, they reached for how `perl` stores it. And they had a genuinely
good objection: "Perl's types" isn't one thing. There are
the sigils, `$` and `@` and `%`, real and enforced at compile time —
you can't `splice` a scalar. There are the storage types, SV and AV and HV.
There's *context*, the thing that decides what `+` does to its operands. There's
the Int/Num/Str business I've been describing. And there are lists, which don't
seem to slot in anywhere. Wasn't I mashing five different things into one pile
and calling it a type system?

I think the answer — and it took me an embarrassingly long time to get here — is
that yes, they're five different things, and the entire trick is giving each one
its right home instead of flattening them into a single tree. Very quickly,
because every one of these deserves its own post:

The sigils are context bound at compile time — `$` versus `@` is the static
version of the exact discrimination context does at runtime. The SV/AV/HV layer
is `perl`'s *representation*, its private type system for storing values, and the
whole load-bearing point of this essay is that it's just one target's choice.
LLVM's representation is `i64` and `double`. `perl`'s is the SV. Turning a Perl
integer into a `perl` IV, or into an LLVM `i64`, is a coercion the
*implementation* picks. It is not a fact about Perl.

Context is the one I find genuinely beautiful, so let me indulge. What does `+`
do to its operands? It demands they be numbers. But that's not a separate
mysterious machine — `+` is a typed function, `(Num, Num) → Num`,
and Perl coerces the arguments to fit, the way any language coerces
arguments at a call boundary. And when a function is *polymorphic* on context —
`reverse` reverses a list in list context and reverses a string in scalar
context —

```perl
my @r = reverse(1, 2, 3);   # (3, 2, 1)
my $s = reverse "hello";    # "olleh"
```

— then context is simply the type it's being dispatched on. `wantarray` is the
language handing a function a mirror so it can see which type it's being asked
for. That's return-type polymorphism, the thing Haskell needs a whole typeclass
mechanism to pull off, sitting right there in Perl's grammar the entire time. We
never called it by its name.

And here's my favorite consequence, the one that made me trust the
framework instead of just liking it. Run the two tests on the *sigils
themselves*. Is a scalar a subtype of a list? A scalar in list context becomes a
one-element list; a one-element list in scalar context is the scalar back again.
Round-trips cleanly. A *multi*-element list forced back into a scalar loses
everything but a count. So the round trip only survives for one-element lists —
which means a scalar *is* a list, specifically the one-element kind. Scalar is a
subtype of List. That quietly demolishes the tidy diagram everyone draws with
Scalar and List sitting side by side as siblings, and — better — it *explains*
something you already know in your hands: assigning a scalar into a list is free
and lossless, and assigning a list into a scalar throws information away.

```perl
my @a = ($x);    # free — widening to the supertype
my $n = @a;      # lossy — narrowing to the subtype, you get a count
```

The framework didn't need me to tell it any of that. It told *me*, and corrected
my own diagram on the way. That's when a model has earned its keep — when it
starts arguing with the person who built it and turns out to be right.

Dualvars end up in exactly the right place too. A dualvar — `$!` is the one
everybody's met, the error variable that's a readable string one way and an
error number the other —

```perl
open my $fh, '<', '/nope' or do {
    warn "message: $!";        # No such file or directory
    warn "number:  ", 0 + $!;  # 2
};
```

— costs nothing in SV-land, because an SV already has a string slot and a number
slot sitting right next to each other. Ask any *other* implementation to
represent it and it turns into a real decision: build a little two-faced struct
for it, or refuse to compile it at all. Same value; free in one representation, a
design choice in another. This whole essay compressed into a single
data type: representation is a separate, per-target type system, not a fact about
the language.

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
