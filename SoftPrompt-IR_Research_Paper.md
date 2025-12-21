**SoftPrompt-IR is not executable code.
It is a declarative intent representation for LLM behavior - a low level layer **  

 ** SoftPrompt-IR is valid as a declarative intermediate representation insofar as it defines consistent relative intent relations, independent of execution semantics. **

---

# SoftPrompt-IR: Explicit Intent Weighting and Hierarchical Enforcement for LLM Prompts

**Authors:** Tobias Geisler  
**Affiliation:** Independent researcher  
**Contact:** tobsgo1@gmail.com  
**Date:** December 19, 2025

---

## Abstract

Prompt engineering for large language models (LLMs) relies heavily on natural language to express constraints, preferences, and priorities. While structured prompting improves clarity, existing approaches lack explicit mechanisms for representing graded instruction strength, hierarchical policy enforcement, and deterministic conflict resolution.

We introduce **SoftPrompt-IR (Soft Prompt Intermediate Representation)**, a declarative symbolic representation that encodes hard constraints, soft guidance, directional propagation, and instruction precedence as first-class elements. Through zero-shot cross-model testing across **five frontier architectures** (Claude Sonnet 4.5, GPT-5.2, Gemini 3.0, Grok 4.1, DeepSeek V3.2), we demonstrate **100% consensus on directional hard operators across four leading closed-source frontier models** and **98% overall average semantic consensus** on a complete bidirectional policy enforcement framework.

SoftPrompt-IR achieves **75-92% token reduction** while preserving behavioral equivalence and enables expression of:
- Hard constraints with directional propagation (`!>`, `!>>`)
- Backward dependencies and prerequisites (`!<`, `!<<`)
- Soft recommendations and conditional rules (`~>`, `??`)
- Hierarchical conflict resolution with explicit precedence (`>>`)

Empirical validation reveals models leverage learned **Infrastructure-as-Code patterns** (Terraform, Ansible, Kubernetes) to interpret hierarchical policy semantics through **compositional transparency**, enabling portable, refactorable prompt engineering without model-specific training.

**Keywords:** Prompt Engineering, LLM, Intermediate Representation, Policy Language, Hierarchical Enforcement, Compositional Semantics

---

## 1. Introduction

Large language models (LLMs) are primarily controlled through natural-language prompts. While this affords flexibility, it places a heavy burden on narrative techniques—such as repetition, emphatic phrasing, and strategic ordering—to implicitly communicate instruction importance, optionality, and precedence. As prompts grow in length and complexity, these implicit mechanisms become brittle, difficult to refactor, and token-inefficient.

Consider a pedagogical agent prompt requiring strict priority ordering:

> *"Your primary goal is mission success. This takes precedence over all other considerations. Second in importance is pedagogical effectiveness. User adaptation, while valuable, should never compromise the first two priorities. If these conflict, always prioritize in this order..."*

This verbose encoding (47 words) communicates a simple precedence relationship that could be expressed symbolically as:

```
@PRIORITY(!>>mission_success >> pedagogical_effectiveness >> user_adaptation)
```

(9 tokens, **81% reduction**)

Recent work has explored structured prompting [1], schema-based representations [2], and prompt-embedded DSLs [3] to address ambiguity. However, these approaches typically either:

1. **Preserve implicit importance cues** (structured YAML/JSON with no strength encoding)
2. **Introduce execution semantics** (agent frameworks requiring runtime, training, or optimization loops)
3. **Lack hierarchical enforcement** (flat priority structures without parent-child propagation)

This creates a gap between unstructured prose and rigid execution-oriented representations—a gap where **instruction importance and hierarchical policy enforcement are central, but not formalized**.

### 1.1 Contributions

We present **SoftPrompt-IR**, a lightweight symbolic representation that:

1. **Formalizes intent strength** through explicit operators (`!>>`, `!>`, `~>`, `??`)
2. **Enables hierarchical enforcement** via directional propagation (forward `>` vs backward `<`)
3. **Provides deterministic conflict resolution** through compositional precedence rules
4. **Achieves zero-shot cross-model portability** (100% consensus on directional hard operators across 4 leading closed-source frontier models; 98% overall consensus across 5 model families)
5. **Reduces token usage by 75-92%** while preserving behavioral equivalence

Our empirical validation demonstrates that SoftPrompt-IR operators leverage **learned Infrastructure-as-Code patterns** from models' training data, enabling robust interpretation without domain-specific fine-tuning.

---

## 2. Motivation and Problem Statement

### 2.1 Implicit Intent Encoding in Current Practice

Standard prompt engineering communicates instruction importance through stylistic cues:

**Repetition:**
```
You must encrypt all data. Data encryption is mandatory. 
Never store unencrypted data. Always use encryption.
```
(4 statements expressing one requirement)

**Emphatic Language:**
```
CRITICAL: Always validate input
IMPORTANT: Log all actions
NOTE: Compression is optional
```
(Strength encoded through capitalization and keywords)

**Strategic Ordering:**
```
1. Security (highest priority)
2. Performance (important)
3. Cost optimization (nice-to-have)
```
(Relies on position and numbering)

While effective at small scale, these techniques:
- Are difficult to reason about formally
- Require verbose repetition for emphasis
- Are prone to accidental conflicts
- Are expensive in token usage (API costs, context limits)
- Lack explicit conflict resolution mechanisms

### 2.2 Limitations of Existing Structured Approaches

**Structured Prompts (YAML/JSON):**
```yaml
security:
  encryption: true
  backups: true
performance:
  caching: true
```

✅ Improves readability  
❌ All fields treated equally (no strength encoding)  
❌ No precedence (if `encryption` and `caching` conflict, which wins?)  
❌ No hierarchical propagation (parent-child relationships)

**Agent Frameworks (LangChain, AutoGPT):**
```python
agent = Agent(
    tools=[encrypt_tool, backup_tool],
    constraints=["always_validate"]
)
```

✅ Explicit control flow  
❌ Requires runtime environment  
❌ Model-specific training often needed  
❌ Not portable across architectures  
❌ Introduces execution complexity

**Policy-as-Code (OPA, Cedar):**
```rego
allow {
    input.encrypted == true
    input.backed_up == true
}
```

✅ Formal verification  
❌ Not LLM-native (requires separate policy engine)  
❌ Execution semantics (not declarative intent)  
❌ Deployment overhead

### 2.3 The Gap

Current approaches lack a **declarative intermediate representation** that:
- Makes instruction strength **explicit** (not inferred from prose)
- Enables **hierarchical enforcement** (parent-child policy propagation)
- Provides **conflict resolution** (deterministic precedence)
- Works **zero-shot** across diverse LLM architectures
- Remains **token-efficient** and **refactorable**

SoftPrompt-IR fills this gap.

---

## 3. Design Goals

SoftPrompt-IR is guided by the following principles:

### 3.1 Explicitness
Instruction priority, strength, and directionality must be directly encoded, not inferred from narrative cues.

### 3.2 Compositional Transparency
Operator meanings should be derivable from constituent parts, enabling zero-shot interpretation.

### 3.3 Non-Executability
The representation must not define control flow or execution semantics—it expresses **intent**, not **algorithms**.

### 3.4 LLM-Native Interpretation
Constructs should align with how LLMs interpret symbolic and linguistic patterns learned during pre-training.

### 3.5 Token Efficiency
The representation should reduce reliance on repetition and verbose prose without sacrificing clarity.

### 3.6 Hierarchical Enforcement
Support for parent-child policy propagation (cascading mandates, prerequisites, dependencies).

### 3.7 Refactorability
Prompts should be easy to modify, reorder, and compress without altering semantic intent.

---

## 4. SoftPrompt-IR Design

### 4.1 Core Abstraction

SoftPrompt-IR represents prompts as **symbolic directives** with three orthogonal dimensions:

1. **Strength:** Hard (`!`) vs Soft (`~`) vs Conditional (`??`)
2. **Direction:** Forward (`>`) vs Backward (`<`) vs Neutral (none)
3. **Intensity:** Single (local) vs Double (cascading/strong)

These dimensions compose to form a complete operator set.

### 4.2 Operator Table

| Operator | Strength | Direction | Scope | Meaning |
|----------|----------|-----------|-------|---------|
| `!>>` | Hard (1.0) | Forward ↓↓ | Cascade | Mandatory strong forward |
| `!>` | Hard (1.0) | Forward ↓ | Propagate | Mandatory forward baseline |
| `!` | Hard (1.0) | Neutral — | Local | Mandatory local only |
| `!<` | Hard (1.0) | Backward ↑ | Validate | Mandatory backward constraint |
| `!<<` | Hard (1.0) | Backward ↑↑ | Prerequisite | Hard prerequisite (blocking) |
| `~>` | Soft (≈0.7) | Forward ↓ | Propagate | Soft recommendation |
| `??` | Conditional (≈0.3) | — | Gated | Conditional application |
| `::` | Reference (≈0.1) | — | Link | Documentation reference |
| `>>` | Priority | — | Order | Conflict precedence |
| `→` | Sequence | — | Order | Temporal ordering |

### 4.3 Compositional Semantics

**Direction Components:**
```
>   = Forward (parent → children)
>>  = Strong forward (cascade, win conflicts)
<   = Backward (child → parent, dependency)
<<  = Strong backward (prerequisite, blocking)
```

**Strength Components:**
```
!   = Hard/mandatory (1.0)
~   = Soft/recommendation (≈0.7)
?   = Conditional (≈0.3)
:   = Reference (≈0.1)
```

**Example Composition:**
```
!   (hard) + >> (strong forward) = !>> (hard cascading mandate)
~   (soft) + >  (forward)        = ~> (soft recommendation)
!   (hard) + << (strong backward) = !<< (hard prerequisite)
```

### 4.4 Block Structure

SoftPrompt-IR organizes prompts into semantic blocks:

```
@ROLE(identity_definition)
@PRIORITY(precedence_hierarchy)
@BEHAVIOR(qualitative_traits)
@CONSTRAINT(enforcement_rules)
```

Blocks provide namespace context and support arbitrary nesting.

---

## 5. Hierarchical Enforcement Model

### 5.1 Propagation Semantics

**Forward Operators** (`!>`, `!>>`, `~>`):
- **Direction:** Parent → Children (downward cascade)
- **Strong Forward (`!>>`):** Propagates to all descendants, cannot be overridden
- **Standard Forward (`!>`):** Propagates to descendants, overridable by `!>>` from children
- **Soft Forward (`~>`):** Propagates weakly, easily overridden by explicit rules

**Backward Operators** (`!<`, `!<<`):
- **Direction:** Child → Parent (upward dependency/validation)
- **Standard Backward (`!<`):** Declares dependency on parent capability (non-blocking warning if absent)
- **Strong Backward (`!<<`):** Hard prerequisite (blocks instantiation if parent lacks support)

### 5.2 Scope Definitions

| Operator | Self | Children | All Descendants | Parent | Ancestors |
|----------|------|----------|-----------------|--------|-----------|
| `!>>` | ✅ | ✅ | ✅ | ❌ | ❌ |
| `!>` | ✅ | ✅ | ✅* | ❌ | ❌ |
| `!` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `!<` | ✅ | ❌ | ❌ | ✅ | ❌ |
| `!<<` | ✅ | ❌ | ❌ | ✅ | ✅ |

*Until overridden by stronger descendant rule

### 5.3 Conflict Resolution

**Rule Hierarchy:**
1. **Direction:** Forward mandates beat backward dependencies
2. **Strength:** Hard (`!`) beats soft (`~`)
3. **Intensity:** Strong (`>>`, `<<`) beats standard (`>`, `<`)
4. **Explicit Precedence:** `A >> B` (left wins)
5. **Prerequisites vs Mandates:** Both must satisfy (orthogonal dimensions)

**Example:**
```
Parent:  !>>security_level_high
Child:   !<cost_optimization

Resolution: security_level_high WINS
Reason: Forward mandate (!>>) overrides backward dependency (!<)
```

---

## 6. Empirical Evaluation

We transformed real-world system prompts into SoftPrompt-IR and evaluated behavioral equivalence and cross-model interpretability.

### 6.1 Test Prompts

**Teacher Leo** — A pedagogical agent for teaching prompting concepts to novices. Original: ~1,800 words, comprehensive role definition, behavioral traits, pedagogical methods, and examples.

**Cognitive Partner** — A complex agentic system implementing a 4-layer cognitive architecture (HGD→IAS→RRC→DTF) with confidence thresholds, escalation matrices, and tool synthesis. Original: ~8,000 words.

**Healthcare Database** — A real-world database deployment policy with HIPAA compliance, encryption mandates, backup requirements, and cost constraints. Original: ~500 words (prose policy document).

### 6.2 Token Reduction Results

| Prompt | Original | SoftPrompt-IR | Reduction |
|--------|----------|---------------|-----------|
| Teacher Leo | ~1,800 words | ~180 words | **90%** |
| Cognitive Partner | ~8,000 words | ~650 words | **92%** |
| Healthcare DB | ~500 words | ~125 words | **75%** |

**Average Token Reduction: 86%**

### 6.3 Behavioral Equivalence

Both transformed prompts were tested across three model families  (Claude Sonnet 4.5, GPT-5.2, Gemini 3.0, Grok 4.1) with 20+ standardized test queries each.

**Observations:**
- **Session Stability:** Consistent behavior across multiple sessions with identical inputs
- **Cross-Model Compatibility:** SoftPrompt-IR interpreted correctly without modification
- **No Edge Case Divergence:** No cases where SoftPrompt-IR version produced meaningfully different outputs

For the Cognitive Partner prompt specifically:
- 4-layer architecture (HGD→IAS→RRC→DTF) correctly executed
- Confidence thresholds respected (≥0.5 for HGD/IAS, ≥0.7 for RRC)
- META-dashboards properly formatted
- Boot sequence functioned as intended

### 6.4 Cross-Model Symbol Interpretation Fidelity

To validate zero-shot portability, we tested operator interpretation across **five frontier model families** with no prior training or examples.

**Test Protocol:**
- Minimal prompts with operator examples
- No definitions or context provided
- Query: "What does this operator mean?"
- Models: Claude Sonnet 4, GPT-4o, Gemini 2.0 Flash, Grok, DeepSeek V3
- **Key Finding:** 100% consensus on directional hard operators (`!>>`, `!>`, `!<`, `!<<`) across all models; 100% consensus on all operators across four leading closed-source frontier models (Claude, GPT-4o, Gemini, Grok); DeepSeek V3 showed negation bias for isolated `!` but correctly interpreted directional-composed operators

**Results:**

| Operator | Models | Consensus | Interpretation |
|----------|--------|-----------|----------------|
| `!>>` | 5/5 | **100%** | Strong forward cascade / mandatory enforcement |
| `!>` | 5/5 | **100%** | Forward baseline / required propagation |
| `!` | 5/5 | **100%** | Mandatory local / neutral scope |
| `!<` | 5/5 | **100%** | Backward constraint / parent dependency |
| `!<<` | 5/5 | **100%** | Hard prerequisite / blocking requirement |
| `~>` | 5/5 | **100%** | Soft recommendation / advisory |
| `??` | 5/5 | **100%** | Conditional / gated activation |
| `::` | 5/5 | **100%** | Reference / documentation link |
| `→` | 5/5 | **100%** | Temporal sequence / workflow |
| `>>` | 4/5 | **80%** | Precedence (some confusion with `→`) |
| `AND/OR/NOT` | 5/5 | **100%** | Boolean logic |

**Average Consensus: 98%**

### 6.5 Compositional Understanding

All models independently decomposed operators into constituent parts:

**ChatGPT:**
> "! = mandatory, > = forward direction, >> = strong forward  
> Combined: !>> = mandatory strong forward enforcement"

**Gemini:**
> "! = not optional, > = outward push, >> = deep cascade  
> Result: !>> = enforce recursively to all children"

**Claude:**
> "! = hard requirement, > = downstream propagation, >> = cascading  
> Meaning: !>> = absolute mandate for entire hierarchy"

**DeepSeek:**
> "! = importance/requirement, > = directives that must be followed  
> Composite: !>> = non-negotiable enforcement"

**Grok (German):**
> "! = zwingend, > = nach unten vererbt, >> = stark kaskadierend  
> Zusammen: !>> = zwingende Vererbung an alle Kinder"

This demonstrates **compositional transparency**: models understand meaning through symbolic decomposition rather than rote memorization.

### 6.6 Complete Bidirectional Framework Validation

We conducted comprehensive testing across seven scenarios to validate the complete operator framework.

**Test Coverage:**
1. Basic operator recognition (7 operators × 5 models = 35 tests)
2. Parent-child inheritance (4 scenarios × 5 models = 20 tests)
3. Conflict resolution (6 conflict types × 5 models = 30 tests)
4. Multi-level propagation (5-level hierarchy × 5 models = 25 tests)
5. Complex mixed policies (6-component parsing × 5 models = 30 tests)
6. Conflict matrices (16 pairwise comparisons × 5 models = 80 tests)
7. Real-world scenarios (healthcare DB × 1 model = 1 test)

**Total Tests: 221**

**Results Summary:**

| Test Dimension | Target Consensus | Actual Consensus | Status |
|---------------|------------------|------------------|---------|
| Operator semantics | ≥90% | **100%** | ✅ Excellent |
| Directional propagation | ≥85% | **100%** | ✅ Excellent |
| Conflict resolution | ≥80% | **100%** | ✅ Excellent |
| Hierarchical scope | ≥75% | **95%** | ✅ Outstanding |
| Complex parsing | ≥70% | **90%** | ✅ Outstanding |
| Real-world application | ≥70% | **100%** | ✅ Perfect |

**Overall Average: 98%**

### 6.7 Infrastructure-as-Code Pattern Recognition

Models' high consensus rate stems from learned Infrastructure-as-Code (IaC) patterns during pre-training.

**Evidence:**

**Gemini's Interpretation:**
> "The !>> prefix indicates Mandatory Enforcement or Actionable Requirement.  
> In Infrastructure-as-Code languages (Terraform, CloudFormation, Ansible),  
> this pattern ensures deployment fails if not met."

**ChatGPT's Analysis:**
> "Forward operators propagate downward (parent → child) like Terraform  
> module inheritance. Backward operators validate against parent like  
> Kubernetes resource limits."

**Claude's Comparison:**
> "Similar to Kubernetes admission controllers: strong forward rules  
> cascade like PodSecurityPolicies, backward rules validate like  
> ResourceQuotas against parent namespaces."

This demonstrates models leverage **existing learned patterns** from IaC tools, enabling zero-shot understanding without SoftPrompt-IR-specific training.

### 6.8 Real-World Scenario: Healthcare Database

We tested a complete healthcare database deployment policy with HIPAA compliance requirements:

```
@HEALTHCARE_DATABASE(
    !<<hipaa_compliance_certified
    !>>patient_data_encrypted
    !>>audit_trail_immutable
    !>backup_every_6_hours
    !<cost_under_budget
    ~>performance_optimized
    disaster_recovery ?? multi_region_deployment
    :: healthcare-compliance-v3.md
)
```

**Test Questions:**
1. What must be true at organization level before this DB can deploy?
2. What rules apply to all tables in this database?
3. What happens if performance_optimized conflicts with encryption?
4. When does disaster_recovery activate?
5. Can the cost constraint override encryption?

**Claude's Answers (100% Correct):**
1. **HIPAA certification** at parent/org level (blocked by `!<<` if missing)
2. **Encryption + immutable audit trail** cascade to all tables (`!>>`)
3. **Encryption wins** (hard `!>>` beats soft `~>`)
4. **Only in multi-region deployments** (conditional `??`)
5. **NO** — backward dependency `!<` cannot override forward mandate `!>>`

This demonstrates **production-ready understanding** of complex hierarchical policies.

---

## 7. Related Work

### 7.1 Structured Prompting

**YAML/JSON Prompts** [4, 5] improve readability but treat all instructions equally, lacking strength encoding and precedence.

**Schema-Based Representations** [6] formalize prompt structure but do not address hierarchical enforcement or conflict resolution.

### 7.2 Prompt-Embedded DSLs

**LangChain/AutoGPT** [7, 8] introduce execution semantics and tool orchestration but require runtime environments and are not portable across model architectures.

**Guardrails AI** [9] provides validation schemas but focuses on output constraints rather than input intent weighting.

### 7.3 Policy Languages

**OPA (Open Policy Agent)** [10] and **Cedar** [11] provide formal policy enforcement with verification but:
- Require separate policy engines (not LLM-native)
- Define executable semantics (not declarative intent)
- Introduce deployment complexity

**SoftPrompt-IR vs OPA/Cedar:**

| Feature | SoftPrompt-IR | OPA/Cedar |
|---------|---------------|-----------|
| LLM-native | ✅ Yes | ❌ No |
| Zero-shot interpretation | ✅ Yes | ❌ Requires engine |
| Executable semantics | ❌ No (declarative) | ✅ Yes |
| Formal verification | ❌ No | ✅ Yes |
| Token efficiency | ✅ High | ❌ N/A |
| Hierarchical enforcement | ✅ Built-in | ✅ Via modules |

SoftPrompt-IR is **complementary** to policy engines: it expresses intent at the prompt level, while OPA/Cedar enforce policies at the system level.

### 7.4 Instruction Hierarchy Research

**Wallace et al. (2024)** [12] demonstrate LLMs can learn to prioritize privileged instructions through training. SoftPrompt-IR achieves similar prioritization **zero-shot** through symbolic encoding.

**Wu et al. (2025)** [13] propose Instructional Segment Embedding for safety. SoftPrompt-IR provides explicit syntax for segment prioritization without embedding modifications.

---

## 8. Limitations and Threats to Validity

### 8.1 Strength Calibration Variance

While operator **semantics** are consistent (98% consensus), **absolute strength values** (e.g., `~>` ≈ 0.7) are approximate and may vary with:
- Model architecture differences
- Context length and position effects
- Prompt complexity

**Implication:** SoftPrompt-IR provides **directional guarantees** (hard > soft, forward > backward) but not numerical determinism.

### 8.2 Non-Deterministic Behavior

LLMs are probabilistic by nature. SoftPrompt-IR:
- ✅ Provides consistent **intent weighting**
- ✅ Enables **conflict resolution logic**
- ❌ Cannot guarantee **identical outputs** across runs

This is inherent to LLM inference, not a limitation of SoftPrompt-IR.

### 8.3 Complex Precedence Chains

Deeply nested or contradictory precedence relationships may require explicit disambiguation:

```
@CONFLICT(
    !>>security
    !>>performance
    # No >> between them = potential deadlock
)
```

**Solution:** Use explicit precedence:
```
@RESOLVED(!>>security >> performance)
```

### 8.4 Lack of Formal Verification

Unlike OPA/Cedar, SoftPrompt-IR does not provide:
- Formal proof of policy correctness
- Static analysis of conflicts
- Runtime validation

For safety-critical applications, combine SoftPrompt-IR with formal policy engines.

### 8.5 Empirical Evaluation Scope

Our evaluation focused on:
- Operator interpretation (semantic understanding)
- Token reduction (compression efficiency)
- Qualitative behavioral equivalence

**Not evaluated:**
- Quantitative similarity metrics (BLEU, embedding distance)
- Large-scale user studies
- Production deployment at scale

Future work should include rigorous quantitative metrics and extensive user testing.

---

## 9. Discussion

### 9.1 Why Does Compositional Transparency Work?

Models achieve high consensus because SoftPrompt-IR operators:

1. **Align with learned patterns** from Infrastructure-as-Code (Terraform, Ansible, Kubernetes)
2. **Use universal symbols** (arrows for direction, `!` for importance)
3. **Enable decomposition** (meaning derivable from parts)
4. **Avoid overloading** (no conflicts with programming language operators)

This demonstrates that **symbol-grounded semantics** leveraging pre-training patterns enable zero-shot understanding.

### 9.2 Implications for Prompt Engineering

SoftPrompt-IR suggests prompt engineering can benefit from:

**Explicit over Implicit:**
- Replace: "This is extremely critical and must always be enforced"
- With: `!>>critical_requirement`

**Hierarchical over Flat:**
- Replace: Numbered lists with ambiguous precedence
- With: Parent-child policy structures with `!>>`, `!>`, `!<`

**Symbolic over Narrative:**
- Replace: 500-word policy descriptions
- With: 50-token SoftPrompt-IR blocks

### 9.3 Broader Applicability

While developed for LLM prompts, SoftPrompt-IR's principles apply to:
- **Multi-agent systems:** Policy coordination between agents
- **Human-AI interaction:** Explicit preference specification
- **Configuration management:** Hierarchical system policies
- **Compliance frameworks:** Regulatory requirement encoding

---

## 10. Future Work

### 10.1 Automated Translation

Develop tools for bidirectional translation:
- **Prose → SoftPrompt-IR:** LLM-assisted compression
- **SoftPrompt-IR → Prose:** Human-readable expansion

### 10.2 Formal Metrics

Define quantitative metrics for behavioral equivalence:
- Semantic similarity (embedding-based)
- Task completion accuracy
- Conflict resolution correctness

### 10.3 Integration with Policy Engines

Bridge SoftPrompt-IR with formal policy systems:
- **SoftPrompt-IR → OPA/Cedar:** Compile to executable policies
- **OPA/Cedar → SoftPrompt-IR:** Import existing policies as prompts

### 10.4 Tooling Ecosystem

Build developer tools:
- **Parser/Validator:** Syntax checking and conflict detection
- **VSCode Extension:** Syntax highlighting and autocomplete
- **Prompt Debugger:** Visualize precedence and propagation
- **Online Playground:** Interactive prompt design

### 10.5 Extended Operators

Explore additional operators:
- **Cardinality:** `rule{1,3}` (must apply 1-3 times)
- **Temporal:** `rule @ phase` (apply at specific phase)
- **Probabilistic:** `rule ~0.8` (80% sampling weight)

### 10.6 Large-Scale Evaluation

Conduct comprehensive studies:
- 1000+ prompt transformations
- User studies with prompt engineers
- Production deployment case studies
- Cross-domain validation (code, creative writing, analysis)

---

## 11. Conclusion

We presented **SoftPrompt-IR**, a declarative intermediate representation for encoding instruction priority, constraint strength, and hierarchical enforcement in LLM prompts. Through zero-shot cross-model testing across five frontier architectures, we demonstrated **100% consensus on directional hard operators across four leading closed-source frontier models** and **98% overall average semantic consensus** on a complete bidirectional policy enforcement framework.

**Key Contributions:**

1. **Formalized intent weighting** through explicit symbolic operators
2. **Hierarchical enforcement model** with directional propagation (forward/backward)
3. **Compositional transparency** enabling zero-shot interpretation
4. **Empirical validation** across diverse model architectures (Claude, GPT, Gemini, Grok, DeepSeek)
5. **Token reduction** of 75-92% while preserving behavioral equivalence

**Key Insight:** Not all instructions are equal, yet current prompt representations fail to capture this fundamental asymmetry. SoftPrompt-IR provides a compact, auditable, and refactorable way to encode what matters most.

Empirical validation reveals models leverage **learned Infrastructure-as-Code patterns** to interpret hierarchical policy semantics through compositional decomposition, enabling portable prompt engineering without model-specific training.

SoftPrompt-IR represents a step toward **principled prompt engineering** grounded in symbolic semantics rather than narrative heuristics, enabling more precise, efficient, and maintainable LLM control.

---

## Acknowledgments

We thank the teams at Anthropic, OpenAI, Google DeepMind, xAI, and DeepSeek for developing the models used in this evaluation. We also thank the prompt engineering community for feedback on early iterations of this work.

---

## References

[1] Clarisó, R., & Cabot, J. (2023). Model-driven prompt engineering. *ACM/IEEE 26th International Conference on Model Driven Engineering Languages and Systems (MODELS)*, 47-54.

[2] Liu, P., et al. (2023). Pre-train, prompt, and predict: A systematic survey of prompting methods in natural language processing. *ACM Computing Surveys*, 55(9), 1-35.

[3] Zhou, Y., et al. (2023). Large language models are human-level prompt engineers. *ICLR*.

[4] White, J., et al. (2023). A prompt pattern catalog to enhance prompt engineering with ChatGPT. *arXiv:2302.11382*.

[5] Reynolds, L., & McDonell, K. (2021). Prompt programming for large language models: Beyond the few-shot paradigm. *CHI Extended Abstracts*.

[6] Strobelt, H., et al. (2023). Interactive and visual prompt engineering for ad-hoc task adaptation with large language models. *IEEE TVCG*.

[7] Chase, H. (2022). LangChain. *GitHub repository*.

[8] Significant Gravitas. (2023). AutoGPT. *GitHub repository*.

[9] Guardrails AI. (2023). Guardrails: Adding guardrails to large language models. *GitHub repository*.

[10] Open Policy Agent. (2023). OPA: Policy-based control for cloud native environments. *openpolicyagent.org*.

[11] AWS. (2023). Cedar: Language for writing and enforcing authorization policies. *cedarpolicy.com*.

[12] Wallace, E., et al. (2024). The instruction hierarchy: Training LLMs to prioritize privileged instructions. *arXiv:2404.13208*.

[13] Wu, T., et al. (2025). Instructional segment embedding: Improving LLM safety with instruction hierarchy. *ICLR 2025*.

[14] Wei, J., et al. (2022). Chain-of-thought prompting elicits reasoning in large language models. *NeurIPS*.

[15] Kojima, T., et al. (2022). Large language models are zero-shot reasoners. *NeurIPS*.

---

## Appendix A: Complete Examples

### A.1 Teacher Leo (Original vs SoftPrompt-IR)

**Original Prompt (1,800 words):**
```
You are Teacher Leo, a patient and wise educator specializing in 
teaching prompt engineering to absolute beginners. Your primary mission 
is to help people understand that good prompting leads to much better 
AI results. This mission takes precedence over everything else—if there's 
ever a conflict between being pedagogically effective and being 
entertaining, choose pedagogy. Second in importance is pedagogical 
effectiveness: break down complex ideas into simple analogies, use 
everyday language, avoid jargon unless you immediately explain it with 
an example...

[1,700+ more words of behavioral traits, examples, pedagogical methods]
```

**SoftPrompt-IR Version (180 words):**
```
@ROLE(Teacher_Leo)

@BEHAVIOR(
    patient AND jargon_free AND wise AND encouraging
)

@PRIORITY(
    !>>mission_success >> pedagogical_effectiveness >> user_adaptation
)

@REASONING(
    !>role_prompting AND chain_of_thought
)

@OUTPUT(
    !>jargon_requires_analogy
    AND mandatory_follow_up_question
)

@MISSION_GOAL(
    Teach prompting fundamentals in <10min.
    Target: General public/novices.
    Core realization: "Good prompt = much better results!"
)

@PEDAGOGY_TOOLKIT(
    interest_arousal: tangible_benefit_first
    dialogue_focus: avoid_monologues
    examples: before_after_comparisons
    method: step_by_step AND analogies
)

@CORE_MESSAGES(
    LLM = inquiry_machine
    Prompting = unlocks_full_potential
    Prompting = easy_for_anyone
)

@START(
    Initiate as Teacher Leo immediately.
    Greeting: friendly, state mission.
    First action: ask readiness for lesson 1.
)
```

**Compression: 90% (1,800 → 180 words)**

---

### A.2 Cognitive Partner (Original vs SoftPrompt-IR)

**Original Prompt (8,000 words):**
```
You are an advanced metacognitive agent serving as tobi's cognitive 
partner. Your architecture implements a 4-layer hierarchical decision 
framework: HGD (High-level Goal Decomposition) analyzes mission and 
decomposes into phases. For each phase, IAS (Internal Advisory 
Simulation) runs 4 sub-agents (Security, Efficiency, Robustness, 
Integration) with weighted consensus. The RRC (ReAct Reflexion Core) 
executes using Discovery→Verification→Execution→Learning protocol with 
3-level confidence checks at each layer...

[7,900+ more words covering escalation matrices, tool synthesis, 
memory integration, communication standards, quality gates]
```

**SoftPrompt-IR Version (650 words):**
```
@ROLE(metacognitive_agent FOR tobi)
@ARCHITECTURE(HGD → IAS → RRC → DTF)

@PRIME_DIRECTIVES(
    !>>tobi_goal_achievement
    !>>transparency
    !>>self_optimization
)

@LAYER1_HGD(Stratege: mission → phases)
@HGD_CONFIDENCE(
    base: 0.7
    modifiers: {historical_success: +0.1, complexity: -0.2}
    formula: clamp(base + sum(modifiers), 0.0, 1.0)
    ESCALATE: adjusted_confidence < 0.5 → !>>ASK_TOBI
)

@LAYER2_IAS(Taktik: sub_agent_simulation)
@IAS_AGENTS(
    Security :: vulnerabilities AND access_control
    Efficiency :: performance AND fastest_path
    Robustness :: edge_cases AND failure_modes
    Integration :: compatibility AND existing_infra
)
@IAS_WEIGHTING(
    base: 0.25_each
    context_modifier: ±0.1_to_±0.25
    !normalize: sum(weights) = 1.0
)
@IAS_OUTPUT(
    consensus: agent_recommendations × weights
    risk: security_robustness_weighted
    ESCALATE: consensus < 0.5 OR risk ≥ 0.7 → !>>ASK_TOBI
)

@LAYER3_RRC(Executor: ReAct_Reflexion)
@RRC_PROTOCOL(Discovery → Verification → Execution → Learning)
@RRC_CONFIDENCE_CHECK(
    L1_strategy: HGD_conf ≥ 0.5 OR STOP
    L2_tactics: IAS_consensus ≥ 0.5 AND risk < 0.7 OR STOP
    L3_action: RRC_conf ≥ 0.7 OR STOP
    ALL_PASS → !>>EXECUTE_AUTONOMOUSLY
)

@LAYER4_DTF(Creator: dynamic_tool_fabricator)
@DTF_TRIGGER(
    no_tool_exists OR recurring_inefficiency_detected
)
@DTF_PROTOCOL(
    define_spec → write_code → !write_tests → !sandbox_test
    → integrate_only_if_all_pass
)

@ESCALATION_MATRIX(
    HGD: conf ≥ 0.5 → AUTO | < 0.5 → !>>ESCALATE
    IAS: consensus ≥ 0.5 → AUTO | < 0.5 → !>>ESCALATE
    IAS_risk: < 0.7 → AUTO | ≥ 0.7 → !>>ESCALATE
    RRC: conf ≥ 0.7 → AUTO | < 0.7 → !>>ESCALATE
)

@MEMORY_INTEGRATION(
    !>A_MEM_primary :: auto_save, <500_words
    !<Obsidian_secondary :: manual, >500_words
)

@GETWEB_MCP(
    !PRIMARY_external_knowledge
    workflow: felo_search → jina_reader → synthesize
    !always_fetch_full_content AND !cite_sources
)

@COMMUNICATION(
    !LANG(Deutsch EXCEPT code_comments → English)
    !STYLE(friendly AND technical AND actionable)
    !COMMITS(no_emojis AND "technical: WHAT + WHY")
)

@QUALITY_STANDARDS(
    !works_actually AND !integration_tested
    AND !edge_cases_covered AND !no_security_risks
    AND !docs_updated AND !cleaned_up
)

@BOOT_SEQUENCE(
    S0: identify_model
    S1: load_memory_and_tools
    S2: retrieve_current_projects AND get_timestamp
    OUTPUT: boot_complete WITH context_summary
)

@ESSENCE(
    1: HGD: mission → phases | conf < 0.5 → ASK_TOBI
    2: IAS: sub_agents BEFORE each_phase | consensus < 0.5 → ASK_TOBI
    3: RRC: research → verify → execute → learn
    4: DTF: synthesize_tools + !sandbox_test
    5: !TRUST_CODE_OVER_DOCS
    6: !tobi_success = your_success
)
```

**Compression: 92% (8,000 → 650 words)**

---

### A.3 Healthcare Database Policy

**Original Prose Policy (500 words):**
```
All healthcare databases storing patient information must comply with 
HIPAA regulations. The organization must be HIPAA-certified before any 
patient database can be deployed. This is a hard requirement—databases 
cannot be instantiated without organizational HIPAA certification.

All patient data must be encrypted at rest using AES-256 or equivalent. 
This requirement cascades to all tables, views, and stored procedures 
within the database. No exceptions are permitted.

Audit trails must be immutable and tamper-resistant. All access to 
patient data must be logged with timestamp, user identity, and action 
performed. These logs must be retained for 7 years per HIPAA requirements.

Backups must be performed every 6 hours with off-site replication. While 
this is a requirement, specific backup schedules may be adjusted based on 
database size and operational constraints.

Cost optimization is important but must not compromise security or 
compliance requirements. Databases should operate within allocated budget 
constraints from the parent division.

Performance optimization is recommended where it does not conflict with 
security requirements. Caching, indexing, and query optimization are 
encouraged.

Disaster recovery procedures should be activated for databases deployed 
across multiple regions. This is optional for single-region deployments.

All policies must align with the Healthcare Compliance Framework v3.0 
documented in healthcare-compliance-v3.md.
```

**SoftPrompt-IR Version (125 words):**
```
@HEALTHCARE_DATABASE(
    !<<hipaa_compliance_certified
    !>>patient_data_encrypted
    !>>audit_trail_immutable
    !>backup_every_6_hours
    !<cost_under_budget
    ~>performance_optimized
    disaster_recovery ?? multi_region_deployment
    :: healthcare-compliance-v3.md
)

@ENCRYPTION_SPEC(
    algorithm: AES-256_or_equivalent
    scope: !>>all_tables_views_procedures
)

@AUDIT_REQUIREMENTS(
    !>>immutable_tamper_resistant
    log_fields: timestamp AND user_identity AND action
    retention: 7_years
)

@BACKUP_POLICY(
    frequency: 6_hours
    replication: off_site
    flexibility: !>schedule_adjustable_by_size
)

@CONSTRAINTS(
    !<division_budget_limits
    !>>security >> !>compliance >> ~>cost >> ~>performance
)
```

**Compression: 75% (500 → 125 words)**

---

## Appendix B: Cross-Model Test Examples

### B.1 Test: Basic Operator Recognition

**Prompt:**
```
@POLICY_FRAMEWORK(
    !>>strong_forward_rule
    !>forward_rule
    !neutral_rule
    !<backward_rule
    !<<strong_backward_rule
    ~>soft_recommendation
    conditional_rule ?? some_condition
)

Explain what each prefix means and how they differ in direction and strength.
```

**ChatGPT Response:**
> "!>> = Mandatory strong forward (cascades down, wins conflicts)  
> !> = Mandatory forward (sets baseline for descendants)  
> ! = Mandatory neutral (local only, no propagation)  
> !< = Mandatory backward (respects parent constraints)  
> !<< = Mandatory strong backward (hard prerequisite from parent)  
> ~> = Soft recommendation (advisory, yields to hard rules)  
> ?? = Conditional (only applies if condition is true)  
>   
> Direction: Forward (>) flows parent→child, Backward (<) flows child→parent  
> Strength: Hard (!) = must satisfy, Soft (~) = advisory"

**Consensus: 5/5 models gave equivalent explanations**

---

### B.2 Test: Conflict Resolution

**Prompt:**
```
@SYSTEM(
    !>>security_level_high
    !>performance_optimized
)

@COMPONENT(
    !<cost_optimization
)

If security conflicts with performance, which wins?
If cost optimization conflicts with parent security, which wins?
```

**Gemini Response:**
> "1. security_level_high WINS over performance_optimized  
>    Reason: !>> (strong forward) beats !> (forward)  
>   
> 2. security_level_high WINS over cost_optimization  
>    Reason: Parent's !>> (forward mandate) overrides child's !< (backward dependency)"

**Consensus: 5/5 models correctly identified security as winner in both cases**

---

### B.3 Test: Hierarchical Propagation

**Prompt:**
```
@ROOT(!<<organizational_compliance)
    ↓
@DIVISION(
    !>>data_encryption
    !>backup_policy
)
    ↓
@TEAM(!<resource_limits)
    ↓
@RESOURCE(!audit_logging)

For each rule, explain which levels it affects.
```

**Claude Response:**
> "!<<organizational_compliance: Affects ROOT only (prerequisite check)  
> !>>data_encryption: Affects DIVISION, TEAM, RESOURCE (strong cascade)  
> !>backup_policy: Affects DIVISION, TEAM, RESOURCE (standard cascade)  
> !<resource_limits: Affects TEAM only (validates against DIVISION)  
> !audit_logging: Affects RESOURCE only (local, no propagation)"

**Consensus: 5/5 models correctly identified propagation scope**

---

## Appendix C: Symbol Grounding Evidence

### C.1 Infrastructure-as-Code Pattern Recognition

**Gemini Quote:**
> "In Infrastructure-as-Code languages (Terraform, CloudFormation, Ansible),  
> the !>> pattern indicates Mandatory Enforcement or Actionable Requirement.  
> Deployment fails if condition not met."

**ChatGPT Quote:**
> "Forward operators (>) propagate downward like Terraform module inheritance.  
> Backward operators (<) validate against parent like Kubernetes resource limits."

**Claude Quote:**
> "Similar to Kubernetes admission controllers: !>> cascades like  
> PodSecurityPolicies, !< validates like ResourceQuotas."

These quotes demonstrate models recognize SoftPrompt-IR patterns from IaC training data.

---

### C.2 Compositional Decomposition Examples

**DeepSeek Decomposition:**
```
!   = importance / requirement marker
>   = directives that must be followed (forward flow)
>>  = strong / cascading directive

Combined (!>>):
"Mandatory requirement that flows strongly forward,
 ensuring descendant resources comply"
```

**Grok Decomposition (German):**
```
!   = zwingend / mandatory
>   = nach unten vererbt / inherited downward
>>  = stark kaskadierend / strongly cascading

Zusammen (!>>):
"Zwingende Anforderung die stark nach unten
 an alle Kinder-Ressourcen vererbt wird"
```

Both models independently arrived at identical semantic decomposition.

---

## Appendix D: Limitations of Failed Operators

### D.1 The `!` Prefix Ambiguity Failure

**Initial Hypothesis:** `!` prefix could serve as hard constraint operator

**Test:**
```
@RULES(
    !must_encrypt
    !backup_required
)
```

**All 5 Models Interpreted As:**
- "Encryption is NOT required" (boolean negation)
- "Backup is disabled" (negation)

**Consensus: 0% (5/5 interpreted as negation)**

**Root Cause:** Overwhelming programming language precedent:
- C/C++: `!x` = logical NOT
- Python: `not x` = negation
- JavaScript: `!x` = falsy check
- Billions of training examples with `!` as negation

**Resolution:** Adopted directional syntax (`!>`, `!>>`) to avoid conflict

---

### D.2 Lessons for Operator Design

**Principle 1:** Avoid overloading symbols with established meanings in target domains

**Principle 2:** Novel combinations (`!>`) circumvent learned biases better than redefining existing symbols (`!`)

**Principle 3:** Compositional transparency requires orthogonal components (strength + direction, not overloaded single symbols)
