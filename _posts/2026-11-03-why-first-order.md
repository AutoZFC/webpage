---
layout: post
title: "Why bring Isabelle/ZF down to first-order logic?"
description: "Deleting the meta-logic turns a proof library into data, and switching from natural deduction to the sequent calculus becomes a program. Can AI treat formulas, not words, as first-class citizens?"
image: /assets/why-first-order-card.png
---

[isa2fol](https://github.com/AutoZFC/isa2fol) translates Isabelle/ZF into plain first-order logic. This post explains
why that is worth doing, and the question it leads to.

## Delete the meta-logic

Most proof assistants put a meta-level on top of the logic you care about. In Isabelle that is Pure, a higher-order
meta-logic with its own ⋀, ⟹ and λ, and above it tactics such as `blast` and `simp`. That stack is great for writing
proofs. It gets in the way when you want to treat proofs as data.

isa2fol removes the meta-level. Every theorem becomes a first-order proof in NBG set theory: three predicates (`elem`,
`eq`, `set`), no function symbols, and NBG's 19 axioms, nothing else admitted.

- Pure's ⟹ and ⋀ become ordinary → and ∀.
- λ-terms, functions and constants become classes.
- A predicate variable becomes a schema, proved once with the predicate left open.

NBG is conservative over ZFC, so nothing changes for statements about sets. A small kernel,
[fol-zfc](https://github.com/AutoZFC/fol-zfc-comb), checks every proof. Today 9,404 of the 9,420 named theorems in
Isabelle/ZF's eight sessions translate and check.

![Isabelle/ZF's four layers, with tactics and the Pure meta-logic removed, become one first-order NBG layer that both ND and LK proofs sit on]({{ "/assets/stack.svg" | relative_url }})

### A taste of reification

With no function symbols, how do you write `succ(i)`, which is i ∪ {i}? You reify it: turn the operation into an
object the logic can talk about, namely its graph. Mathematicians know the trick already, since a function *is* a set
of pairs.

- `succ` becomes a class `Succ` of input–output pairs, and the term `succ(i)` becomes "the r with ⟨i, r⟩ ∈ `Succ`".
- So `x ∈ succ(i)` becomes, roughly, `∃r. ⟨i, r⟩ ∈ Succ ∧ x ∈ r`.
- Each such class owes two facts, proved once from the axioms: it exists, and it gives exactly one r per input.
- Several arguments are packed into one tuple first, and a λ-term is reified the same way, right where it appears.

If you write compilers, this is defunctionalization: a function becomes data, and applying it becomes a membership
lookup.

## What it buys: a century of logic, as programs

First-order logic is the most studied logic there is. Once the whole library is plain first-order logic, its classic
theorems stop being facts on paper and become translators you can run on every proof:

- **Sequent calculus.** Gentzen (1935) turns natural deduction into LK, the calculus built for search.
- **SAT.** Herbrand's theorem reduces a first-order proof to a propositional tautology over finitely many instances,
  which is what SAT solvers check.
- **λ-calculus.** Curry–Howard reads natural-deduction proofs as typed λ-terms (λμ for classical steps), so
  simplifying a proof is running a program.
- **Automated provers.** First-order logic is the native input of provers such as Vampire and E, so every statement
  is a ready-made benchmark.

Over a meta-logic full of tactics, these theorems have nothing to hold on to. Over first-order logic, each one is a
program, and only the small kernel has to be trusted, not the program.

The first of these is built. Natural deduction (ND) is good for reading proofs; LK is good for searching for them,
because every rule works backwards from the goal. In theory the translation covers every proof. In practice fol-zfc's
NDtoLKR converts 99.8% of a 3,236-claim sample with no translator errors; the other 0.2% ran out of memory. An
unmodified LK checker verifies every output. Counting each identity step (A ⊢ A) as one step, an LK proof is 1.08×
the size of its ND proof.

## The question: formulas, not words

AI provers today treat mathematics as text. Tokens go in, tokens come out, and a checker accepts or rejects them
afterwards. Most token strings are not formulas, so the model spends effort learning grammar it could have been given.

Yet most of this library is search, not writing: 83% of Isabelle/ZF's proofs contain no human-written term or formula.

So: can we build symbolic AI that treats well-formed formulas, not words, as first-class citizens? Encode formulas by
their structure, act only with legal inference steps, and search. A chess engine sees a board and its legal moves, not
the move list as text. First-order logic, with the meta-logic gone, gives mathematics a board.
