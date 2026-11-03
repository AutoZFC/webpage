---
layout: post
title: "Why bring Isabelle/ZF down to first-order logic?"
description: "Deleting the meta-logic turns a proof library into data, and classic results about first-order logic become tools you can run on it. Can AI treat formulas, not words, as first-class citizens?"
image: /assets/why-first-order-card.png
---

[Isabelle/ZF](https://isabelle.in.tum.de/) is the set theory library of the Isabelle proof assistant: thousands of
machine-checked theorems, from ordinals and cardinals to Gödel's constructible universe.
[isa2fol](https://github.com/AutoZFC/isa2fol) translates it into plain first-order logic. This post explains why that
is worth doing, and the question it leads to.

## Delete the meta-logic

Most proof assistants put a meta-level on top of the logic you care about. Isabelle's is a higher-order meta-logic
(called Pure) with its own "implies" ⟹, "for all" ⋀ and λ. Above it sit tactics, proof automation commands such as
`blast` and `simp`. That stack is great for writing proofs. It gets in the way when you want to treat proofs as data.

isa2fol removes the meta-level. Every theorem becomes a first-order proof in NBG (von Neumann–Bernays–Gödel), a
standard set theory that also has classes. There are three predicates (membership, equality and "is a set"), no
function symbols, and nothing assumed beyond NBG's 19 axioms.

- The meta-level ⟹ and ⋀ become ordinary → and ∀.
- Functions, constants and anonymous functions (λ-terms) become classes.
- A theorem about an arbitrary property P is proved once with P left open, then instantiated wherever it is used.

NBG proves exactly the same theorems about sets as ZFC, the usual foundation of mathematics. A small, separate proof
checker checks every translated proof. Today 9,404 of the 9,420 theorems in Isabelle/ZF translate and check.

![Isabelle/ZF's four layers, with tactics and the meta-logic removed, become one first-order set theory that both natural deduction and sequent calculus proofs sit on]({{ "/assets/stack.svg" | relative_url }})

### A taste of reification

With no function symbols, how do you write `succ(i)`, which is i ∪ {i}? You reify it: turn the operation into an
object the logic can talk about, namely its graph. Mathematicians know the trick already, since a function *is* a set
of pairs.

- `succ` becomes a class `Succ` of input–output pairs, and the term `succ(i)` becomes "the r with ⟨i, r⟩ ∈ `Succ`".
- So `x ∈ succ(i)` becomes, roughly, `∃r. ⟨i, r⟩ ∈ Succ ∧ x ∈ r`.
- Each such class comes with two facts, proved once from the axioms: it exists, and it gives exactly one r per input.
- Several arguments are packed into one tuple first, and an anonymous function is reified the same way, right where
  it appears.

If you write compilers, this is defunctionalization: a function becomes data, and applying it becomes a membership
lookup.

## What it buys: classic results become tools

First-order logic is the most studied logic there is. Many of its classic theorems are proved by a recipe: give me a
proof in one form, and I will build it in another. Once the whole library is plain first-order logic, each recipe can
be written as a tool and run on every proof:

- **Sequent calculus.** Gentzen (1935): rewrite a natural-deduction proof, the usual style of written proofs, as a
  sequent-calculus (LK) proof, a form built for search.
- **SAT.** Herbrand's theorem: reduce a first-order proof to a propositional formula over finitely many instances,
  which a SAT solver can check.
- **λ-calculus.** Curry–Howard: read a natural-deduction proof as a typed program (with an extension for classical
  reasoning), so that simplifying the proof means running the program.
- **Automated provers.** Provers such as Vampire and E take first-order logic as input, so every statement in the
  library is a ready-made benchmark.

These recipes are stated for first-order proofs, so a meta-logic full of tactics gives them nothing to hold on to.
And the tools need not be trusted: the small checker checks everything they produce.

The first of these tools is built. Natural deduction is good for reading proofs; the sequent calculus is good for searching
for them, because every rule works backwards from the goal. In theory the translation covers every proof. In practice
our translator converts 99.8% of a sample of 3,236 proofs with no errors; the other 0.2% ran out of memory. A separate
sequent-calculus checker verifies every result. The new proofs come out about the same size as the originals (1.08×),
once each routine "A follows from A" check counts as a single step.

## The question: formulas, not words

AI provers today treat mathematics as text. Tokens go in, tokens come out, and a checker accepts or rejects them
afterwards. Most token strings are not formulas, so the model spends effort learning grammar it could have been given.

Yet most of this library is search, not writing: 83% of Isabelle/ZF's proofs contain no human-written term or formula.

So: can we build symbolic AI that treats well-formed formulas, not words, as first-class citizens? Encode formulas by
their structure, act only with legal inference steps, and search. A chess engine sees a board and its legal moves, not
the move list as text. First-order logic, with the meta-logic gone, gives mathematics a board.
