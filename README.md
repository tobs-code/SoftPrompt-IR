[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

## SoftPrompt-IR

### Low-Level Intent Weighting & Direction Annotation for LLM Prompts

SoftPrompt-IR is **not** a DSL, not a framework, and not a policy system.

It is a **minimal symbolic annotation layer** for explicitly expressing:

* intent **strength**
* intent **direction**
* intent **priority**
* intent **dominance / override**

inside natural language prompts.

It defines **no execution logic**.
It only exposes **intent structure** that is otherwise implicit.

---

## Why SoftPrompt-IR Exists

LLMs already reason over latent signals such as:

* priority
* strength
* hierarchy
* enforcement vs. avoidance
* dominance between instructions

Today, these signals are usually:

* implicit
* buried in prose
* distributed across sentences
* easy to contradict unintentionally
* hard to inspect or debug

SoftPrompt-IR makes these signals **explicit and localized**.

Not to gain more control —
but to **reduce ambiguity, noise, and unintended behavior** *before sampling*.

---

## Core Idea

SoftPrompt-IR uses **small symbolic operators** to annotate intent along two axes:

1. **Strength** (how much it should matter)
2. **Direction** (enforce vs. avoid, downstream vs. blocking)

It does **not** tell the model *what to do* —
it tells the model **what matters more than what**.

Think of it as **weighting signals**, not commands.

---

## Minimal Example

```text
@TASK(
  "Write a short story about a failed mission."
)

~<<< PURPLE_PROSE
~<<  CLICHES
~<   OVER_EXPLAINING
````
These are relative avoidance signals, not prohibitions.

### Interpretation (informal)

* Strongly avoid purple prose
* Moderately avoid clichés
* Gently discourage over-explaining

There are:

* no hard rules
* no prohibitions
* no forced decoding

Only **probability shaping** via relative weight.

---

## Cascading & Dominance (The Important Part)

SoftPrompt-IR is intentionally **asymmetric**.

More symbols = more weight
Stronger markers dominate weaker ones

```text
!>>> PRIMARY_CONSTRAINT
~>>  SECONDARY_PREFERENCE
~>   OPTIONAL_STYLE
```

This creates a **cascade**:

* strong intent dominates weaker intent
* global signals dominate local ones
* conflicts resolve by weight, not prose order

No if/else.
No branching.
Just dominance.

More weight only matters relative to other signals — stacking strength everywhere removes contrast and reduces its effect.
---

## Direction Matters

Direction is part of the signal — not decoration.

In SoftPrompt-IR, **direction never stands alone**.
It always modifies a **strength marker**.
(Direction never modifies intent on its own — it always attaches to an intent marker.)

### Canonical Form

```
[STRENGTH][DIRECTION]
```

Strength **always comes first**.
Direction **always comes after**.

---

### Valid Examples

```text
!>>> MUST_ENFORCE
~<<< STRONGLY_AVOID
~<   GENTLY_DE_ESCALATE
```

Informal intuition:

* `>` pushes intent forward / downstream
* `<` pulls intent away / de-escalates
* repetition amplifies directional influence
* strength determines enforcement level

---

### Dependencies (Special Case)

Dependencies are **explicit prerequisite signals**

They are defined **only** like this:

```text
!<<< REQUIRED_CONDITION
```

Meaning:

> This intent **cannot proceed unless** the requirement is satisfied.

Dependencies are **not equivalent** to “strong backward preference”.
They are a **separate semantic construct**.

---

### Invalid Forms (Do Not Use)

```text
>>! SOME_CONSTRAINT
<<~ SOMETHING
```

Why these are invalid:

* strength must not appear after direction
* direction must not precede strength
* there is no postfix weighting in SoftPrompt-IR

---

These signals are **not formal semantics** —
they are **structural cues the model already recognizes** from rules, configs,
policies, and priority annotations seen during training.

---

## Why This Works (Without Training)

LLMs have seen massive amounts of:

* config files
* rulesets
* policies
* flags
* priority markers
* logs and diagnostics

SoftPrompt-IR does **not** invent meaning.

It leverages the model’s ability to exploit:

* consistent structure
* relative salience
* symbolic hierarchy

---

## What SoftPrompt-IR Is *Not*

SoftPrompt-IR is **not**:

* a jailbreaking technique
* a prompt hacking method
* a safety bypass
* a classifier replacement
* a programming language

In many cases, it does the opposite:
👉 it makes unsafe or contradictory intent **harder to hide**.

---

## Why Low-Level Matters

SoftPrompt-IR operates **below** common prompt conventions.

That makes it:

* composable with any prompt
* compatible with safety systems
* ignorable if unsupported
* predictable when it fails

Low-level primitives tend to:

* age well
* combine easily
* degrade gracefully
* be hard to misuse accidentally

---

## Design Philosophy

SoftPrompt-IR prioritizes:

* explicit over implicit
* weighting over wording
* de-escalation over prohibition
* structure over cleverness
* compatibility over control

---

**SoftPrompt-IR is meant to annotate only the parts that tend to be misweighted, not every detail of a prompt.**

---

## Status

This is an **experimental specification**.

There is:

* no enforcement
* no guarantee
* no hidden mechanism

Just a small idea:

> Make intent **visible** before sampling.

If it feels obvious in hindsight —
that means it’s probably at the right level 🙂
