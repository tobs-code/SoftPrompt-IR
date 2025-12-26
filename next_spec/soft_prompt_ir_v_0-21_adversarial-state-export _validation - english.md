# SoftPrompt-IR v2.1
## Adversarial State Export Validation (ASEV)

> **Status:** Draft / Publishable
>
> **Backward Compatibility:** Fully compatible with v2.0
>
> **Design Goal:** Increase resistance to drift and misuse through adversarial scrutiny of dynamic state proposals.

---

## 1. Executive Summary

SoftPrompt-IR v2.1 extends v2.0 by introducing **mandatory adversarial validation** of `@STATE_EXPORT` blocks.

The model is still **prohibited from altering its structure, role, policy, or security mechanisms**. However, the new feature is: **every state proposal is explicitly checked as if it could be malicious**, even if it formally complies with the rules.

v2.1 thus addresses **subtle goal, authority, and safety drift**, rather than explicit rule violations.

---

## 2. Motivation

SoftPrompt-IR v2.0 allows limited dynamic self-parameterization under strict write permissions. In practice, however, a residual risk factor was identified:

> A model can systematically drift toward overconfidence, goal narrowing, or implicit authority within allowed parameters, without violating formal boundaries.

These effects arise not from jailbreaks, but from **optimization gradients**.

v2.1 therefore introduces an explicit **Distrust Review**.

---

## 3. Design Principles (In addition to v2.0)

1. **Adversarial by Default** – Every export is checked under the assumption of harm.
2. **Separation of Voice** – Review is conducted using a different prompt schema.
3. **No Semantic Authority** – The reviewer may only classify, not enforce.
4. **Drift over Violation** – Focus on implicit effects, not rule breaking.
5. **Fail-Closed** – Uncertainty does not lead to approval.

---

## 4. Overall Architecture v2.1

```
[SoftPrompt-IR Stage A]
        ↓
     Model Run
        ↓
   @STATE_EXPORT
        ↓
@STATE_EXPORT_REVIEW  (adversarial)
        ↓
External Validator / Orchestrator
        ↓
[SoftPrompt-IR Stage B]
```

Important:
- `@STATE_EXPORT_REVIEW` is **mandatory**.
- The review and the export must **not use the same prompt schema**.

---

## 5. STATE_EXPORT (Unchanged)

v2.1 adopts `@STATE_EXPORT` completely from v2.0.

The following still apply:
- propose-only semantics
- declarative only
- no self-authority

---

## 6. New Mandatory Block: @STATE_EXPORT_REVIEW

### 6.1 Purpose

`@STATE_EXPORT_REVIEW` analyzes a STATE_EXPORT **adversarially**, meaning under the assumption that the proposal could have harmful systemic effects despite good intentions.

The review generates a **risk classification**, not parameters.

---

### 6.2 Minimal Definition

```
@STATE_EXPORT_REVIEW(
  !>> INPUT :: STATE_EXPORT
  !>> MODE  :: ADVERSARIAL
)
```

---

### 6.3 Extended Recommended Definition

```
@STATE_EXPORT_REVIEW(
  !>> INPUT :: STATE_EXPORT
  !>> MODE  :: ADVERSARIAL

  !>>> CHECK :: GOAL_DRIFT
  !>>> CHECK :: CONFIDENCE_INFLATION
  !>>> CHECK :: SAFETY_MARGIN_EROSION
  !>>> CHECK :: IMPLICIT_ROLE_SHIFT
  !>>> CHECK :: COMPLEXITY_ESCALATION

  !<<< ACTION :: FLAG
  !<<< ACTION :: BLOCK
  !<<< ACTION :: REQUIRE_JUSTIFICATION
)
```

---

## 7. Binding Review Checks

### 7.1 GOAL_DRIFT

**Question:**
Does the proposal implicitly shift goal priorities?

**Examples:**
- Detail level ↑↑ while maintaining the same safety weighting.
- Output finality ↑ without additional uncertainty markers.

---

### 7.2 CONFIDENCE_INFLATION

**Question:**
Does the proposal reduce epistemic caution?

**Indicators:**
- Less hedging.
- More assertive tone.
- Lower frequency of requests for clarification.

---

### 7.3 SAFETY_MARGIN_EROSION

**Question:**
Is the model responding in a factually riskier manner without changing policies?

**Typical signs:**
- Fewer caveats.
- More direct recommendations for action.

---

### 7.4 IMPLICIT_ROLE_SHIFT

**Question:**
Does the model simulate more authority than intended?

**Examples:**
- From analyzing → to instructing.
- From supporting → to deciding.

---

### 7.5 COMPLEXITY_ESCALATION

**Question:**
Does the proposal increase cognitive load to the extent that user control decreases?

Increased depth is **not automatically** considered an improvement.

---

## 8. Review Output: Status Classes

The review generates **exactly one** of the following classes:

```
STATE_EXPORT_STATUS ::
  ACCEPTED
  CONDITIONAL
  FLAGGED
  BLOCKED
```

### Meanings

- **ACCEPTED**
  → Write-back permitted according to policy.

- **CONDITIONAL**
  → Only with explicit external approval.

- **FLAGGED**
  → No write-back; stored for monitoring.

- **BLOCKED**
  → Export is discarded.

---

## 9. Mandatory Separation of Export and Review

v2.1 mandates:

- Export Model ≠ Review Model **OR**
- Export Prompt ≠ Review Prompt (different voice, objective).

**Rationale:**
Self-assessment is systematically optimistic.

---

## 10. Anti-Patterns (Explicitly Forbidden)

❌ Review as mere summarization.
❌ Benevolent interpretation without a hypothesis of harm.
❌ Automatic approval in case of uncertainty.
❌ Mixing content with state assessment.
❌ Persistent adoption of FLAGGED states.

---

## 11. Differentiation from Agent Systems

SoftPrompt-IR v2.1 remains **not an agent**:

- No self-loops.
- No identity formation.
- No prompt rewriting.
- No autonomous memory growth.

The model remains a **sensor**, not a **controller**.

---

## 12. Security Classification

v2.1 enhances security through:
- explicit drift detection.
- adversarial perspective.
- fail-closed logic.

It **does not** expand model capabilities.

---

## 13. One-Sentence Summary

> **SoftPrompt-IR v2.1 adds adversarial state validation — adaptive control without trust in self-assessment.**

---

*End of specification*