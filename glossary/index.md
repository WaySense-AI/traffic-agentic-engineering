---
layout: default
title: "Glossary"
nav_order: 30
---

# Glossary

**Status: normative.** This is the payload of the specification.

The chapters argue a position. This page defines vocabulary. If TAE is ever cited, it will most likely be cited for one of the terms below — which is why this page and the [primitive specification](../primitives/index.md) are licensed [CC BY 4.0](./LICENSE.md) while the prose is not.

Terms are ordered by dependency: later entries assume earlier ones.

---

## Traffic Agentic Engineering (TAE)

The discipline of building AI systems that carry **engineering responsibility** in traffic domains — systems that produce deliverables which can be traced, validated, and defended.

TAE's central claim is that traffic engineering is a **compilation problem**, not a knowledge problem. Adding model capability does not close the gap between "the system can discuss a queue length" and "the system is accountable for one." Closing it requires a formal architecture: responsibility → IR → planner → primitive → dependency graph → compiler → validation → delivery.

> Ch. 2 · See also: [Engineering Responsibility](#engineering-responsibility-er)

## Engineering Responsibility (ER)

**The atomic unit of traffic engineering.**

A responsibility is an assignment that carries: a defined scope, an accountable party, required evidence, applicable constraints, completion criteria, and produced artifacts. It is distinct from a *task* (which is an action) and from a *workflow* (which is a sequence).

| | Task / Workflow | Responsibility |
|---|---|---|
| Unit | An action to perform | An outcome to answer for |
| Ends when | Steps are executed | Completion criteria are satisfied |
| Failure | Step did not run | Criteria unmet — regardless of effort |
| Accountability | Implicit | Explicit |

This distinction is why a chatbot cannot become an engineer: it can execute steps and produce text, but it holds no responsibility and therefore cannot fail one.

> Ch. 3

## Responsibility Pyramid

The hierarchical decomposition of a responsibility into progressively finer sub-responsibilities, terminating in responsibilities that a single primitive can satisfy.

The pyramid is what makes a responsibility *tractable*: it converts "optimize this corridor" into a set of leaf responsibilities, each of which has explicit evidence requirements and can be independently validated.

> Ch. 3

## World State / World Context

The state of the traffic world that a responsibility is evaluated against: network topology, signal plans, demand, incidents, geometry, and applicable regulatory context.

Recorded explicitly in the IR as **Section 2: World Context**. Two agents given the same responsibility but different world states are answering different questions — so world state is part of the specification, not ambient context.

> Ch. 3, Ch. 4

## Engineering Intermediate Representation (IR)

A **declarative** statement of an engineering responsibility — the layer between intent expressed in natural language and execution as a primitive dependency graph.

The IR is modeled deliberately on compiler IR: a stable, inspectable, technology-independent representation that separates *what must be established* from *how it is computed*.

Six sections:

| # | Section | States |
|---|---------|--------|
| 1 | Responsibility | What is being asked, and the accountable party |
| 2 | World Context | The state it is evaluated against |
| 3 | Required Evidence | What must be measured or established |
| 4 | Constraints | Applicable standards, bounds, regulations |
| 5 | Deliverables | The artifacts that constitute completion |
| 6 | Confidence Policy | Acceptable uncertainty, and what to do when exceeded |

The IR is the reason a TAE system can be **audited**: the transformation from question to result passes through a representation a human engineer can read and disagree with.

> Ch. 4

## Reasoning Planner

The component that expands an Engineering IR into a **reasoning graph** — determining which evidence is needed, which primitives can establish it, and how those primitives depend on one another.

The planner does **not** execute. It produces a plan; the runtime selects operators and executes. The planner asks *"which primitive can establish the evidence I need, given my constraints and quality requirements?"* — never *"which API should I call?"*

> Ch. 5

## Reasoning Graph

The output of planning: a graph of primitive invocations connected by typed dependencies, with the evidence each edge carries made explicit.

Reasoning graphs are the unit of **reproducibility**. Expressed in primitives rather than operators, the same graph can be re-executed against different implementations and the results compared.

> Ch. 5, Ch. 7

## Engineering Primitive

**The smallest unit of executable engineering meaning.**

A primitive is a stable engineering concept — queue length, control delay, lane capacity, saturation degree — that can be established by many different computational methods.

The defining test: if a new algorithm appears next year that computes this quantity better, the primitive is unchanged and only its default operator changes. If the concept itself would change, it was not a primitive.

Primitives are named by engineering meaning in the form `traffic.<domain>.<metric>`, never by implementation:

```
traffic.queue.length        ✓  engineering meaning
CTMQueueCalculator          ✗  implementation
```

> Ch. 6 · See [Primitive Specification](../primitives/index.md)

## Operator

A concrete computational realization of a primitive.

For `traffic.queue.length`, known operators include cell transmission models, microsimulation (SUMO/VISSIM), video analytics, loop detector statistics, radar, and neural traffic foundation models. Each has different data requirements, accuracy profiles, and costs — and each establishes the *same* engineering meaning.

The primitive/operator split is what lets a TAE system adopt new perception and simulation technology without rewriting its reasoning layer.

> Ch. 6.3

## Traffic Primitive Specification (TPS)

The semantic contract every primitive must expose: identity, semantics, interface, constraints, execution, and quality. TPS describes **reasoning interfaces**, not function signatures.

Full schema: [TPS →](../primitives/index.md)

> Ch. 6.4

## Primitive Dependency Graph (PDG)

A reasoning graph where nodes are primitives and edges are typed engineering dependencies.

TAE recognizes five dependency types:

| Type | Edge means |
|------|-----------|
| **Semantic** | One primitive's output is a required input concept of another |
| **Evidence** | One primitive's result validates or corroborates another |
| **Constraint** | One primitive's output bounds another's valid domain |
| **Temporal** | Ordering matters in time (before/after, peak/off-peak) |
| **Spatial** | Ordering or adjacency matters in space (upstream/downstream) |

Dependencies are **declarative**: they encode engineering knowledge about how quantities relate, not execution order. Execution order is the runtime's problem, and may differ from the declared dependency structure under parallelization.

> Ch. 7

## Evidence / Evidence Contract

Evidence is the measurable, verifiable support for an engineering conclusion. Every primitive must declare **which engineering evidence it can establish** — its evidence contract.

This field is the single most important part of the TPS. A primitive that cannot say what it proves is not usable in engineering reasoning, because there is no way to determine whether a conclusion was supported or merely plausible.

Evidence is what separates validation from verification: verification asks "was it computed correctly?", evidence asks "does it actually support the claim?"

> Ch. 6.5, Ch. 9

## Engineering Delivery

The production of a validated, structured, actionable **artifact** that closes the engineering loop.

> **The Delivery Imperative**: in software engineering, code that is not shipped is code that does not matter. In traffic engineering, reasoning that is not delivered is reasoning that does not improve anyone's commute.

Delivery is what distinguishes an engineering intelligence system from a conversational assistant with domain knowledge. Chat produces answers; engineering produces deliverables.

> Ch. 8

## Deliverable / Artifact

The structured output that satisfies a responsibility's completion criteria: a timing plan, a review report, a diagnostic finding, a compliance determination.

Deliverables are evidence-based — each claim traceable to the primitives that established it and the operators that computed them. This is what makes a deliverable **signable**.

> Ch. 8

## Engineering Validation

The process that converts a generated result into a decision someone will stand behind.

Validation subsumes verification. A result can be computed correctly and still be invalid — wrong method for the context, applied outside its applicability bounds, or unsupported by the evidence collected.

TAE defines a validation pyramid spanning four scopes (primitive, reasoning, strategy, delivery) across ordered levels, beginning with syntax and constraint checking.

> Ch. 9 · See also: [Evidence](#evidence--evidence-contract)

## Confidence Policy

The declared tolerance for uncertainty in a responsibility, and the required behavior when that tolerance is exceeded.

Recorded in the IR as **Section 6**. A system that silently produces low-confidence output is not engineering — the policy makes "I don't know, and here is what I'd need to know" a first-class outcome rather than a failure mode.

> Ch. 4

## Constraint

A bound on valid engineering reasoning: a standard (HCM, MUTCD, local design codes), a physical limit, a regulatory requirement, or a client specification.

Constraints live in the IR (**Section 4**) rather than being baked into primitives, so the same primitive can serve jurisdictions with different standards — and so the applicable standard is auditable.

> Ch. 3, Ch. 4

---

## Terms deliberately excluded

Precision also means saying what TAE does **not** claim:

| Not a TAE term | Why |
|----------------|-----|
| "Traffic brain" | Marketing language. No defined semantics; no completion criteria. |
| "Copilot" | Describes interaction style, not accountability. A copilot holds no responsibility. |
| "Digital twin" | Describes representational fidelity, not engineering reasoning. Orthogonal concern. |
| "End-to-end AI" | In direct tension with TAE: untraceable inference cannot produce signable deliverables. |

---

**License**: [CC BY 4.0](./LICENSE.md) — reuse freely, including commercially, with attribution.
