---
layout: default
title: "TAE Specification"
nav_order: 1
---

# Traffic Agentic Engineering

### An open specification of how traffic agents should think, reason, and be held accountable

**v0.1 — Draft.** Volumes I and II complete — Chapters 1–20.

This is a specification, not a book. It defines concepts precise enough that two independent teams reading it should build interoperable traffic agents — or at minimum, argue productively about where they disagree.

[中文版 →](../zh-CN/index.md) · [Glossary →](../glossary/index.md) · [Primitives →](../primitives/index.md)

---

## The gap

Twenty years of Intelligent Transportation Systems made traffic **observable** and **simulable**. It never made traffic engineering **executable**.

Our models perceive congestion, forecast demand, and simulate corridors at high fidelity. The engineering process is unchanged: engineers still collect data, diagnose, calculate timing plans, and write reports. The computer became intelligent. The engineering did not.

Large language models close none of this gap alone. An LLM can discuss a queue length. It cannot be accountable for one. It produces text that *reads* like engineering judgment without producing the evidence chain that makes engineering judgment defensible — and in traffic engineering, **defensibility is the product**. A timing plan that cannot be traced to a method, a standard, and a measurement cannot be signed.

## The thesis

Traffic engineering is not a knowledge problem that better models will solve. It is a **compilation problem**.

The unit of traffic engineering is not the question and not the task. It is the **engineering responsibility** — an assignment with defined scope, constraints, required evidence, completion criteria, and an accountable party.

Once you accept that, the stack follows:

```
Responsibility  →  IR  →  Reasoning Planner  →  Primitive
      →  Dependency Graph  →  Compiler  →  Validation  →  Delivery
```

## Volume I — Chapters

### Part I · Foundations

| | Chapter |
|---|---------|
| 1 | [The End of Intelligent Transportation?](./chapter-01-the-end-of-intelligent-transportation.md) |
| 2 | [From Traffic Engineering to Traffic Agentic Engineering](./chapter-02-from-traffic-engineering-to-tae.md) |

### Part II · The architecture

| | Chapter |
|---|---------|
| 3 | [Engineering Responsibility — The Atomic Unit of Traffic Engineering](./chapter-03-engineering-responsibility.md) |
| 4 | [Engineering Intermediate Representation](./chapter-04-engineering-intermediate-representation.md) |
| 5 | [Reasoning Planner — Compiling Engineering Thinking](./chapter-05-reasoning-planner.md) |
| 6 | [Engineering Primitive — The Smallest Unit of Executable Reasoning](./chapter-06-engineering-primitive.md) |
| 7 | [Primitive Dependencies — The Grammar of Engineering Reasoning](./chapter-07-primitive-dependencies.md) |
| 8 | [Engineering Delivery — Where Reasoning Creates Value](./chapter-08-engineering-delivery.md) |
| 9 | [Engineering Validation — From Generated Results to Trusted Decisions](./chapter-09-engineering-validation.md) |

## Volume II — Chapters

| | Chapter |
|---|---------|
| 10 | [Engineering Analysis Strategy](./chapter-10-engineering-analysis-strategy.md) |
| 11 | [Domain Reasoning Compiler](./chapter-11-domain-reasoning-compiler.md) |
| 12 | [Engineering Delivery Engine](./chapter-12-engineering-delivery-engine.md) |
| 13 | [Engineering Validation](./chapter-13-engineering-validation.md) |
| 14 | [Traffic Situation Assessment](./chapter-14-traffic-situation-assessment.md) |
| 15 | [Traffic Engineering Strategies](./chapter-15-traffic-engineering-strategies.md) |
| 16 | [Domain Reasoning Compiler — Advanced Mechanisms](./chapter-16-drc-advanced-mechanisms.md) |
| 17 | [Domain Primitive Library — The Building Blocks of Engineering Reasoning](./chapter-17-domain-primitive-library.md) |
| 18 | [Traffic Agent Runtime — Executing Engineering Reasoning](./chapter-18-traffic-agent-runtime.md) |
| 19 | [Engineering Delivery System — From Reasoning to Actionable Outcomes](./chapter-19-engineering-delivery-system.md) |
| 20 | [WayMind — A Reference Implementation of Traffic Agentic Engineering](./chapter-20-waymind-reference-implementation.md) |

## Start here, depending on who you are

| You are | Read |
|---------|------|
| **Evaluating whether TAE matters** | [Glossary](../glossary/index.md), then Ch. 1–2 |
| **Implementing a traffic agent** | [Primitives / TPS](../primitives/index.md), then Ch. 3–4, 6–7 |
| **Worried about trust and auditability** | Ch. 8–9, and [open issues](../primitives/index.md#open-issues) |
| **Skeptical** | Ch. 2.7 (the definition of TAE), then the [terms deliberately excluded](../glossary/index.md#terms-deliberately-excluded) |

## Status and honesty

This is a draft with **known open issues, published rather than hidden** — see [Open issues](../primitives/index.md#open-issues). The most significant: the `category` field in the TPS schema is defined with one value set and exercised with another. It is unresolved, and we would rather publish it as an open question than quietly pick one and pretend it was designed.

Volume II carries the argument through to the compiler, the runtime, and a working reference implementation. Where the earlier chapters say what a component *is*, Volume II says what it *must do* — contracts and guarantees, stated precisely enough that an independent implementation can be checked against them.

## Licensing

| Content | License |
|---------|---------|
| Prose — chapters, argument | [CC BY-NC-SA 4.0](./LICENSE.md) |
| Glossary, TPS, primitive identifiers | [CC BY 4.0](../LICENSE-SPEC.md) |

The asymmetry is deliberate. We are not licensing revenue out of a specification. We are trying to make the vocabulary unavoidable.

---

*Traffic engineering is not a knowledge problem. It is a compilation problem.*
