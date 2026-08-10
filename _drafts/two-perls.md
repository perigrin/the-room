# Two Perls

*Working outline for the prose — the spine, not the draft itself yet. Voice to be punched up by perigrin.*

**Venue:** The Room. Follow-up to the TPRC 2026 talk *"Perl, Agentic Programming, and the Tools We Need"* — fills in the type-system aside mentioned in passing there. Concept note lives in the commonplace book: `pages/two-perls.md`.
**Length target:** ~2000–2800 words. Body + epilogue.
**Voice:** chris.prather.org register — analogy-forward, conversational. Draft gets us close; perigrin punches it up.

## The distinction the whole piece rests on

- **Perl** = the language. What values *mean*. Implementation-independent.
- **`perl`** = one implementation. What the interpreter *stores* and *does*.
- Typography is load-bearing: `Perl` vs `perl` held apart throughout. The distinction *is* the argument.

## Thesis line (working — keep verbatim, punch up later)

> An SV carries the machinery for every possible future a Scalar might have. SSA is a decision to only worry about *right now*. There should be no way to bridge them — but Perl has a latent static type system that gets us there.

## Refrain

> If we didn't have `perl`, what kind of implementation could we have?

Pose at the top. Answer twice at the end: a fast Perl, yes — but really, *the ability to ask the question at all.*

## Arc — body

1. **Two Perls, the foreclosure (open + refrain).** Perl the language vs `perl` the implementation, never pried apart — "only `perl` can parse Perl," "Perl is whatever `perl` does." Not tidy philosophy, a cage: if Perl just *is* `perl`, then `perl` is the only Perl there can be. **So: if we didn't have `perl`, what could we have?**
2. **What `perl` forecloses.** Not optimization — `perl` already optimizes, up to its own per-op SV-dispatch ceiling (`pp_add` unboxing SVs on every `+`). What's off the table is the *common modern kind*: an SSA-based optimizing compiler, and with it **Julia's move** — infer concrete types into LLVM for native speed. A dynamic language can get there; Julia is the existence proof. Perl can't, while a value means an SV.
3. **Why `perl` blocks it — the Katamari SV.** `perl`'s answer to "what is a value" is the SV: machinery for every possible future a scalar might have — mutable, ref-counted, dynamically retypeable, a universal ball of magic. Overkill for SSA, which only worries about *right now*. Designed for a world of mutation/aliasing/retyping; SSA is the world that deleted all three. While "Perl value" means "SV," the fast Perl stays impossible.
4. **The paradox + thesis.** SV (every future) vs SSA (only now) — should be no bridge. But *Perl* — the language, not `perl` — has a latent static type system that gets us there. (Thesis line here.)
5. **Recovering it — the licensed bridge.** The two tests; "is `"42"` a number? *maybe*"; the behavioral contracts are the *proof* the SV's open futures don't happen *here* → licence to drop to `i64`. Where they can't be ruled out, keep the SV. A **licensed** bridge, value by value — not universal. SV features map one-for-one to the guards (mutable→no-aliasing, ref-counted→lifetime-not-value, retypeable→type-is-fixed, magic→no-magic/overload/tie). The SV is a checklist of everything you must rule out; the contracts are that same checklist from the language side.
6. **The shape (rjbs's five, offstage, tight).** Value lattice with ephemeral tops and storable leaves; `Scalar <: List` correcting the naive sibling tree; context = generic dispatch; DualVar free in SV-land, a decision anywhere else. Gesture; let the formal doc carry depth.
7. **Remove `perl` — refrain answered.** The SSA/LLVM target can't fall back to an SV, so it can't cheat — the latent static types must carry the *whole* bridge or it fails, and the failure is the map. "If we didn't have `perl`" → an implementation that knows what Perl *is*, and can therefore make it fast. Description-of-Perl and bridge-to-fast are one artifact.

## Epilogue

Speed was one answer — and not even the main one. Types make development easier everywhere (Python, TypeScript, Ruby, Go, Rust); that's old news. The fresh part for Perl: the work you do to go *fast* is *defining what correct behavior even is*, and once the compiler knows that, the language can finally tell the **developer** "this doesn't make sense" instead of silently DWIMing nonsense. Speed and kindness-to-the-developer fall out of one artifact.

The **two Katamaris**: the SV balls up every possible *future* for the machine; without a type system, the *developer* balls up every value they run across, sensible or not. The same latent static type system un-balls both.

And the point underneath all of it: while Perl means only "what `perl` executes," we can't even have the *conversation* about what Perl is — `perl`'s implementation keeps getting in the way (someone always reaches for an SV). Speed was just the case concrete enough to make it undeniable. That conversation-we-couldn't-have is the tool the talk was really asking for.

## Closing button

The irony: **TIMTOWTDI** — there's more than one way to do it — for the language that has allowed itself exactly *one* way to *be done*: `perl`. The most pluralist language ever made, an implementation monoculture. Reclaiming "more than one way" at the level of *implementations* — to run, to reason about, to define Perl — is the whole project. Larry's motto, finally pointed at `perl` itself.

---

## The model (toolkit for the drafter)

### rjbs's five (offstage — appear as "the natural objections," unattributed), each given a home
1. **sigil / storage types** (`$ @ %`) — compile-time; the static form of context. `splice $x` fails for the same reason `+` coerces: an op demanding a type system of an operand declared in another.
2. **runtime data types** (SV/AV/HV) — `perl`'s *representation* type system, one target among possible targets. Evidence it's separate from Perl's: one SV carries Str, Num, Int, **and** DualVar — many latent types, one representation.
3. **value context** (what `+` does) — operators are typed generics over the lattice (`+ : (Num,Num)→Num`); "numeric context" is just implicit argument coercion to the signature. ③ and ④ are the same thing.
4. **Int/Num/Str TCs** — the latent value lattice (the round-trip+behavior kernel). Scope honestly: only *some* type constraints are transformation-safety assertions; arbitrary predicate TCs (regex, `->where`) are a superset layered on the kernel.
5. **lists** — an **ephemeral** type (see below); the concept that un-flattens the category error.

### Three type systems
- **Perl's latent types** (what a value means) — invariant.
- **Each target's representation type system** — `perl`'s SV/IV/PV; LLVM's i64/double/ptr; C-native. Separate, per-implementation. `Int → IV` and `Int → i64` are *coercions the implementation chooses*.
- Joined by coercions the compiler must **license**.

### Value lattice (by the two tests)
`None <: { Undef, Bool, Int <: Num <: Str, Ref <: {ScalarRef, ArrayRef, HashRef, CodeRef, Object}, DualVar, Regex, Glob } <: Scalar <: List`
- **Ephemeral tops** (`Scalar`, `List`, `None`) — no storage of their own; only handled transiently in context.
- **Storable leaves** — the concrete types variables actually hold.
- **Ephemeral = uncommitted supertype with no storage form.** `Scalar` is *only ever* ephemeral: a stored value is always narrowed to a subtype. As a *static type*, `Scalar` is the "top / not-yet-narrowed" annotation (the `Parm → TOP` case) — same uncommitted top, two binding times.

### Scalar <: List (by the two tests — corrects the naive sibling tree)
- Round-trip: any scalar `$s → ($s) → $s` (singleton, lossless). ✓ Behavioral: singleton satisfies list ops. ✓ → **Scalar <: List**.
- Strict: multi-element and empty lists don't round-trip back → `List ⊀ Scalar`. The scalars are exactly the length-≤1 lists.
- Ephemeral types are **arity classes**: None(0) <: Scalar(1) <: List(n). Subtype direction *predicts* the coercion asymmetry: `@a = ($x)` free upcast; `$s = @a` lossy downcast.

### Context = generic dispatch / return-type polymorphism
- Context is the type a generic function dispatches on. Monomorphic ops (`+`) → just arg coercion; polymorphic (`reverse`, `localtime`) → context selects the instance.
- List/scalar context = return-type polymorphism (Haskell-typeclass-shaped, baked into the grammar). `wantarray` = reflecting on the discriminator. `reverse` is the multimethod proof (list: reverse a list; scalar: reverse a string).

### DualVar
- A promoted *representation* fact that became a genuine Perl type (constructible via `dualvar()`, seen in `$!`). Once semantic-level, invariant. **Every target must implement it in its own type system or explicitly exclude it** — free in SV-land (both slots already there), a struct-or-exclusion in i64-land. Same fact as the guard `DualVar ∉ Int/Num`.

### The two tests (from the formal doc)
1. **Syntactic preservation** (round-trip): `C(v) ≡ id_S(v)`.
2. **Semantic fulfillment** (behavioral contracts).
Both required. (Distinguishes membership from mere coercion — a hashref stringifies but isn't a Str.)

## Examples to use
- Is `42` a number? **yes.** `"hello"`? **no.** `"42"`? **maybe.** — *when does it matter?*
- `1 + 2` is NOT trivially runtime-free: `+` on Int-*typed SVs* still carries SV semantics (magic, overload, tie, dualvar, int/float promotion).
- `reverse` (list vs scalar) — context selects the instance.
- `$!` / `dualvar(404, "Not Found")` — two faces, one value.
- `@a = ($x)` free vs `$s = @a` lossy — the subtype direction derives the asymmetry.

## Honest edges (present, don't fake-close)
- **Scalar↔List fixed point** — the formal doc leaves it an unformalized circular dependency (Knaster–Tarski conjecture). The `Scalar <: List` result straightens it into an ordering, but uniqueness is still conjectured. Present as the honest edge.
- **TC scope** — formalizing the transformation-safety kernel, not all type constraints.

## Sources (primary — local copies in scratchpad/chalk-grab, originals in chalk repo docs/)
- `docs/architecture/perl-type-system-formal.md` — axioms, lattice, soundness (75K).
- `docs/architecture/perl-type-system-practical.md` — the doc rjbs reviewed.
- `docs/plans/2026-06-06-three-axis-codegen-and-typed-ir-contract.md` — latent ≠ representation; unboxing guards = contract violators; LLVM as forcing function.
- `docs/architecture/typed-ir-representation.md` — finalized typed-IR contract.
- gist `2-perl-types-formal.md` (perigrin) — summary of the formal treatment.
