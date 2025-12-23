# SoftPrompt-IR: A Beginner's Guide

You don't need to understand everything here.
You can start using SoftPrompt-IR after the first example.

## The Problem: LLMs Can't Read Your Mind

When you write a prompt like this:

> *"I want a book that's mainly sci-fi and a little bit scary but not too much, cool cars are important but not as important as having a female hero, maybe something about the color blue but that's optional, same with lollipops but those matter more than blue, and absolutely NO love stories under any circumstances..."*

**How should an LLM know:**
- What's mandatory vs. nice-to-have?
- Which "optional" thing is more optional than another?
- What's a hard blocker vs. a soft preference?

It can't. It guesses. And guessing means inconsistent results.

---

## The 60-Second Rule (Read This First)

If you're new, remember just this:

1. Put things you **MUST have** with `!>>>`
2. Put things you **WANT** with `~>`
3. Put things you **DON'T WANT** with `<<<`

That's enough to start.

---

## The Solution: Stop Implying, Start Weighting

SoftPrompt-IR replaces vague words with **explicit visual weight**.

**The confused prompt above becomes:**

```
@GOAL(
  !>>> BOOK
)

@GENRE(
  !>>> SCIENCE_FICTION
  ~>   HORROR
  <<<  LOVE_STORY
)

@CHARACTERS(
  !>>> FEMALE_PROTAGONIST
)

@ELEMENTS(
  ~>>  COOL_CARS
  ~>   LOLLIPOPS
  ~<   COLOR_BLUE
)

@CONTEXT(
  !> USER :: example_user
)
```

Now the intent is **unambiguous**:
- Sci-fi and female hero are **non-negotiable**
- Love story is **blocked** (hard blocker: "absolutely forbidden")
- Cool cars matter more than lollipops, which matter more than blue
- All soft preferences (`~`) can be dropped if needed

---

## The Operators: Your Visual Vocabulary

### The Core Principle

**Two dimensions determine every operator:**

1. **Hard vs Soft:** `!` or no prefix = hard / `~` = soft
2. **Strength:** More arrows = stronger intensity

That's it. Everything else follows from this.

---

### Hard Forward Operators (`!`) — Non-Negotiable Requirements

| Operator | Strength | Meaning |
|----------|----------|---------|
| `!>>>` | Strongest | **Absolute requirement** — cascades to everything below |
| `!>>` | Strong | **Must have** — strong enforcement |
| `!>` | Normal | **Required** — standard rule |

**Example:**
```
@SECURITY(
  !>>> ENCRYPTION          ← Everything must be encrypted, no exceptions
  !>>  AUTHENTICATION      ← Strong requirement
  !>   LOGGING             ← Required, but less critical
)
```

---

### Soft Forward Operators (`~`) — Preferences

| Operator | Strength | Meaning |
|----------|----------|---------|
| `~>>>` | Strongest | **Strongly preferred** — try hard to include |
| `~>>` | Strong | **Preferred** — include if possible |
| `~>` | Normal | **Nice to have** — optional enhancement |

**Example:**
```
@STYLE(
  ~>>> CONCISE             ← Really want this
  ~>>  EXAMPLES            ← Would be great
  ~>   HUMOR               ← If it fits, sure
)
```

---

### Hard Backward Operators — Blockers vs Prerequisites

Backward operators work in **two different ways** depending on whether they have a `!` prefix:

#### Pure Blockers (no `!` prefix) — They forbid things

| Operator | Strength | Meaning |
|----------|----------|---------|
| `<<<` | Strongest | **Absolutely forbidden** — hard stop, no exceptions |
| `<<` | Strong | **Not allowed** — strong block |
| `<` | Normal | **Must avoid** — hard block |

**Example:**
```
@FORBIDDEN(
  <<<  VIOLENCE            ← Absolutely no violence, ever
  <<   PROFANITY           ← No swearing
  <    NEGATIVE_TONE       ← Must avoid negative tone
)
```

#### Prerequisites (with `!` prefix) — They require things

| Operator | Strength | Meaning |
|----------|----------|---------|
| `!<<<` | Strongest | **Strong prerequisite** — absolutely requires this |
| `!<<` | Strong | **Hard prerequisite** — strongly requires this |
| `!<` | Normal | **Prerequisite** — requires this |

**Example:**
```
@REQUIREMENTS(
  !<<< DATABASE_READY      ← Absolutely requires database to be ready
  !<<  API_AVAILABLE       ← Hard prerequisite: API must be available
  !<   CACHE_WARMED        ← Prerequisite: cache should be warmed
)
```

**Key difference:** Blockers (`<<<`) forbid things. Prerequisites (`!<<`) require things.

### Advanced Note (skip on first read): Prerequisites vs Blockers

#### ⚠️ CRITICAL CONCEPT CLARIFICATION: Backward Operators with `!`

**The `!` prefix completely changes what backward arrows mean:**

**Without `!` (Blockers) — They forbid things:**
- `<<< VIOLENCE` = "violence is absolutely forbidden"
- `<< PROFANITY` = "profanity is not allowed"
- `< NEGATIVE_TONE` = "must avoid negative tone"

**With `!` (Prerequisites) — They require things:**
- `!<<< DATABASE_READY` = "absolutely requires database to be ready"
- `!<< API_AVAILABLE` = "hard prerequisite: API must be available"
- `!< CACHE_WARMED` = "prerequisite: cache should be warmed"

**Why this design?** The `!` doesn't "negate" — it switches the semantic meaning from "block this" to "require this". This prevents LLM confusion between prohibitions and requirements.

**Example of correct usage:**
```
@PREREQUISITES(
  !<<< DATABASE_READY     ← Strong prerequisite: absolutely requires database
  !<<  API_AVAILABLE      ← Hard prerequisite: strongly requires API availability
  !<   CACHE_WARMED       ← Prerequisite: requires cache to be warmed
)
```

**Wrong usage (would create confusion):**
```
@FORBIDDEN(
  !<<< VIOLENCE          ← WRONG! This creates a prerequisite, not a prohibition
)
```

**Remember:** Backward arrows without `!` = blockers. With `!` = prerequisites.

**If you're new: you can ignore dependencies for now. Most beginners only need forward rules (`>`) and blockers (`<`).**

---

### Soft Backward Operators (`~`) — Avoidances

These point **backwards** but are **soft** — avoid if possible, but not a hard stop.

| Operator | Strength | Meaning |
|----------|----------|---------|
| `~<<<` | Strongest | **Strongly prefer to avoid** |
| `~<<` | Strong | **Prefer to avoid** |
| `~<` | Normal | **Weakly avoid** — if possible |

**Example:**
```
@PREFERENCES(
  ~<<< TECHNICAL_JARGON    ← Really try to avoid jargon
  ~<<  LONG_SENTENCES      ← Prefer shorter sentences
  ~<   PASSIVE_VOICE       ← Avoid if possible, but okay if needed
)
```

---

### Reference Operator (`::`) — Binding & Context

The `::` operator **binds** something to a value or reference. Think of it like an electrical connection — it's either connected or it's not.

**Critical Rule:** `::` always needs a strength operator in front of it!

Why? Because the binding itself is neutral — the **strength operator determines how critical that connection is**.

| Syntax | Meaning |
|--------|---------|
| `!>>> X :: value` | X is **absolutely bound** to value (non-negotiable) |
| `!>> X :: value` | X is **strongly bound** to value |
| `!> X :: value` | X is **bound** to value (required) |
| `~> X :: value` | ❌ **Doesn't make sense** — "optionally connected"? |

Think of it like wiring: you can't have a "maybe 5V or maybe not" connection. The wire is either there or it isn't. The strength operator defines how critical that wire is to the system.

**Example:**
```
@CONTEXT(
  !>>> USER :: example_user            ← User is definitely example_user
  !>>  LANGUAGE :: german              ← Language is strongly bound to german
  !>   DOMAIN :: software_development  ← Domain is bound to software dev
)

@CONFIG(
  !>>> SOURCE :: company-style-guide.md   ← Config source is absolutely this file
  !>>  SCHEMA :: openapi-spec.yaml        ← Schema is strongly bound to this spec
)
```

---

## Visual Weight = Intuitive Priority

The more arrows, the stronger the weight. Your brain already gets this:

```
HARD FORWARD (requirements)     SOFT FORWARD (preferences)
!>>>  ████████████  (maximum)   ~>>>  ▓▓▓▓▓▓▓▓  (strongly preferred)
!>>   ████████      (strong)    ~>>   ▓▓▓▓▓     (preferred)
!>    █████         (normal)    ~>    ▓▓▓       (nice to have)

HARD BACKWARD (blockers)        SOFT BACKWARD (avoidances)
<<<   ████████████  (forbidden) ~<<<  ▓▓▓▓▓▓▓▓  (strongly avoid)
<<    ████████      (not allowed) ~<<   ▓▓▓▓▓     (prefer to avoid)
<     █████         (must avoid)  ~<    ▓▓▓       (weakly avoid)
```

---

## Real-World Example: Code Assistant

**Before (natural language):**
> *"You're a coding assistant. Always write clean code. Security is the top priority, but performance matters too, just not as much. Use TypeScript when possible. Never expose API keys. Try to add comments but don't overdo it. Follow our style guide."*

**After (SoftPrompt-IR):**
```
@ROLE(
  !>>> CODING_ASSISTANT
)

@PRIORITIES(
  !>>> SECURITY
  !>>  CLEAN_CODE
  !>   PERFORMANCE
)

@LANGUAGE(
  ~>>  TYPESCRIPT
)

@CONSTRAINTS(
  <<<  EXPOSED_SECRETS
)

@STYLE(
  !>>> SOURCE :: company-style-guide.md
  ~>   COMMENTS
)
```

**What changed:**
- Priority is explicit (security > clean code > performance)
- TypeScript is preferred, not required
- Exposed secrets are **absolutely forbidden** (`<<<`), not just "please don't"
- Style source is **hard bound** to the guide file
- Adding comments is a separate **soft preference**

---

## Quick Reference Card

```
HARD FORWARD (requirements)     SOFT FORWARD (preferences)
!>>>  absolute must             ~>>>  really want
!>>   must have                 ~>>   would like
!>    required                  ~>    nice to have

HARD BACKWARD (blockers)        SOFT BACKWARD (avoidances)
<<<   absolutely forbidden      ~<<<  strongly avoid
<<    not allowed               ~<<   prefer to avoid
<     must avoid                ~<    weakly avoid

PREREQUISITES (with !)
!<<<  strong prerequisite       ~<<<  avoid (no ! prerequisite)
!<<   hard prerequisite         ~<<   prefer to avoid
!<    prerequisite              ~<    weakly avoid

BINDING/CONTEXT
!>>> X :: value    absolutely bound
!>>  X :: value    strongly bound
!>   X :: value    bound (required)
```

---

## The Pattern

Once you see it, you can't unsee it:

| | **Hard** | **Soft** |
|---|---|---|
| **Forward (requirements/preferences)** | `!>>>` `!>>` `!>` | `~>>>` `~>>` `~>` |
| **Backward (blockers/avoidances)** | `<<<` `<<` `<` | `~<<<` `~<<` `~<` |
| **Prerequisites (requirements with !)** | `!<<<` `!<<` `!<` | *(no soft prerequisites)* |

- **`!` or no prefix = hard** (must/must not)
- **`~` = soft** (prefer/prefer not)
- **`>` = forward** (requirements)
- **`<` = backward** (blockers)
- **More arrows = stronger**

---

## Why This Works

LLMs have seen millions of config files, infrastructure-as-code (Terraform, Kubernetes, Ansible), and policy documents during training. They already understand:

- Arrows = direction and flow
- More symbols = more weight
- `!` = important/mandatory
- `~` = optional/soft

SoftPrompt-IR doesn't teach LLMs something new — it speaks a language they already know.

---

## TL;DR

1. **`!` or no prefix = hard / `~` = soft**
2. **`>` = forward (requirements) / `<` = backward (blockers)**
3. **More arrows = stronger weight** (`>>>` > `>>` > `>`)
4. **`::` = binding (always needs a strength operator in front!)**
5. **No implicit words** — the structure IS the meaning

   You never write words like "always", "never", "if", or "prefer". The arrows already say that.

---

**Now imagine asking an LLM to guess your intent from prose... or just telling it explicitly.**

Which would you trust more?