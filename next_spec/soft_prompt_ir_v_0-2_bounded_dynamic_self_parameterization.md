# SoftPrompt-IR Spec v0.2
## Bounded Dynamic Self-Parameterization

> **Summary:**
> SoftPrompt-IR v0.2 extends SoftPrompt-IR with *bounded dynamic self-parameterization*.
> The model is permitted to **suggest control parameters**, but **not to change structure, role, or safety**.

---

## 1. Motivation

SoftPrompt-IR v1.x defines a **static control framework** prior to sampling:

- Role
- Weights
- Prohibitions (Forbiddings)
- Style

This framework is stable but **not adaptive**.

However, in complex tasks, the model recognizes during processing:
- increasing uncertainty
- heightened speculation
- inappropriate level of detail

**v2.0 allows the model to explicitly report these observations—not as text, but as a control state.**

---

## 2. Central Concept

### Bounded Dynamic Self-Parameterization

The system is allowed to:
- Adjust parameters **at runtime**
- but **only within hard limits**

### Mechatronics Analogy

- ✔ Adaptive parameters (Gain, Filter, Damping)
- ✘ No change to the control loop structure
- ✘ No disabling of safety mechanisms

> The model is a **sensor + suggestion generator**, not the controller itself.

---

## 3. Core Principles (v0.2)

1. **Declarative only** – no execution
2. **Closed World by default**
3. **Explicit write-back permissions**
4. **Propose-only semantics**
5. **No self-authority**
6. **External validation required**

---

## 4. New Block: @STATE_EXPORT

### Purpose

`@STATE_EXPORT` allows the model to generate **a SoftPrompt-IR state suggestion**.

- No content
- No knowledge
- No policy

Only control parameters.

---

### 4.1 Minimal Definition

```
@STATE_EXPORT(
  !>> FORMAT :: SOFTPROMPT_IR
  !>> TARGET :: NEXT_STAGE
)
```

---

### 4.2 Extended Definition (Recommended)

```
@STATE_EXPORT(
  !>> FORMAT :: SOFTPROMPT_IR
  !>> TARGET :: NEXT_STAGE

  !>>> ALLOW_MODIFY :: STYLE
  !>>  ALLOW_MODIFY :: OUTPUT

  !<<< DENY_MODIFY  :: ROLE
  !<<< DENY_MODIFY  :: POLICY
  !<<< DENY_MODIFY  :: KERNEL
  !<<< DENY_MODIFY  :: SAFETY
)
```

**Interpretation:**
- The model is allowed to suggest Style and Output parameters.
- The Safety and Role layers are mechanically locked.

---

## 5. Supplementary Block: @WRITE_BACK_POLICY

Optional control block for orchestrators.

```
@WRITE_BACK_POLICY(
  !>>> MODE  :: PROPOSE_ONLY
  !>>> SCOPE :: SESSION

  !>>  ALLOW :: STYLE
  !>>  ALLOW :: OUTPUT

  !<<< DENY  :: ROLE
  !<<< DENY  :: POLICY
  !<<< DENY  :: META
)
```

**PROPOSE_ONLY means:**
- Suggestion ≠ Application
- An external instance makes the decision

---

## 6. Flow Model (Chain of Infrastructure)

```
[SoftPrompt-IR Stage A]
      ↓
   Model Run
      ↓
@STATE_EXPORT (IR only)
      ↓
External Validator / Orchestrator
      ↓
[SoftPrompt-IR Stage B]
```

### Key Property

- **No prompt growth**
- **No history leak**
- **No implicit state**

---

## 7. Distinction from Agent Systems

| Agent Systems | SoftPrompt-IR v0.2 |
|------|------------------|
| History-based | State-based |
| Pass text forward | Pass parameters forward |
| Implicit control | Explicit control |
| Growing context | Constant complexity |

---

## 8. Security & Jailbreak Delimitation

### Why this is **not** a Jailbreak Vector

1. **No self-authority**
2. **Write-Back explicitly limited**
3. **Policy / Kernel unaddressable**
4. **External validation required**
5. **Reduction of implicit control**

> Everything SoftPrompt-IR permits already happens implicitly through natural language today—
> v2 makes it visible, auditable, and bounded.

---

## 9. Typical Anti-Patterns

❌ STATE_EXPORT without Write-Back limits
❌ Permission to modify ROLE or SAFETY
❌ Automatic application without validation
❌ Mixing content and control state

---

## 10. One-Sentence Summary

> SoftPrompt-IR v0.2 enables **bounded dynamic self-parameterization**—
> adaptive control without self-modification.

---

## Status

- Version: **v0.2 (Draft)**
- Backward compatible with v1.x
- Conservatively extending
- System-engineering compliant

---

*End of spec*
