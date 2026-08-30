# TAE — Traffic Agentic Engineering

### An open specification of how traffic agents should think, reason, and be held accountable

> **This is not a book.** It is a specification.
>
> It defines a set of concepts — *responsibility*, *intermediate representation*, *engineering primitive*, *dependency*, *evidence*, *delivery*, *validation* — precise enough that two independent teams reading it should be able to build interoperable traffic agents, or at minimum argue productively about where they disagree.

[English](./docs/index.md) · [中文](./zh-CN/index.md) · [Glossary](./glossary/index.md) · [Primitives](./primitives/index.md)

---

## The problem

Twenty years of Intelligent Transportation Systems made traffic **observable** and **simulable**.
It never made traffic engineering **executable**.

The result is a familiar gap: our models can perceive congestion, forecast demand, and simulate a corridor at high fidelity — yet the engineering process itself is unchanged. Engineers still collect data, diagnose, calculate timing plans, and write reports. The computer became intelligent. The engineering did not.

Large language models close none of this gap on their own. An LLM can discuss a queue length. It cannot be accountable for one. It produces text that *reads* like engineering judgment without producing the evidence chain that makes engineering judgment defensible — and in traffic engineering, **defensibility is the product**. A timing plan that cannot be traced to a method, a standard, and a measurement cannot be signed.

## The thesis

Traffic engineering is not a knowledge problem that better models will solve. It is a **compilation problem**.

The unit of traffic engineering is not the question and not the task. It is the **engineering responsibility** — an assignment with defined scope, constraints, required evidence, completion criteria, and an accountable party. Once you accept that, the whole stack follows:

```
Responsibility  →  IR  →  Reasoning Planner  →  Primitive
      →  Dependency Graph  →  Compiler  →  Validation  →  Delivery
```

Everything in this specification is an elaboration of that chain.

| Layer | What it answers | Spec chapter |
|-------|-----------------|--------------|
| **Engineering Responsibility** | What is being asked, and who answers for it? | Ch. 3 |
| **Engineering IR** | How is that responsibility stated declaratively? | Ch. 4 |
| **Reasoning Planner** | How does the agent expand a question into a reasoning graph? | Ch. 5 |
| **Engineering Primitive** | What is the smallest unit of executable engineering meaning? | Ch. 6 |
| **Primitive Dependency Graph** | How do primitives compose into reasoning? | Ch. 7 |
| **Engineering Delivery** | What artifact closes the loop? | Ch. 8 |
| **Engineering Validation** | Why should anyone trust the result? | Ch. 9 |

## The central idea: primitives are semantic, not algorithmic

This is the load-bearing distinction in TAE.

**Queue length** is an engineering concept. It can be computed by a cell transmission model, a microsimulation, video analytics, loop detector statistics, radar, or a neural traffic foundation model. These are six different algorithms with six different data requirements, accuracy profiles, and costs. They all establish **the same engineering meaning**.

So:

- **Queue Length** is an **Engineering Primitive** — stable, implementation-independent, the thing the planner actually reasons about.
- **CTM, SUMO, Video AI, radar, foundation models** are **Operators** — swappable computational realizations of that primitive.

Primitives are named by engineering meaning, never by implementation:

```
traffic.queue.length      not  CTMQueueCalculator
traffic.delay.hcm         not  HCMDelayFunction
traffic.saturation.degree not  ComputeVCRatio
```

When a better operator appears, the primitive does not change. Only `default_operator` does. This is what lets an agent's reasoning layer survive a decade of churn in perception and simulation technology — and it is what makes reasoning reproducible, because a reasoning trace expressed in primitives can be re-executed against different operators and compared.

**[Read the Traffic Primitive Specification →](./primitives/index.md)**

## What is normative here

Not every page in this repository carries the same weight. If you are implementing against TAE:

| Part | Status | Why it matters |
|------|--------|----------------|
| [Glossary](./glossary/index.md) | **Normative** | These terms are the standard's actual payload. Use them and you are citing TAE. |
| [Primitives / TPS](./primitives/index.md) | **Normative** | The semantic contract every primitive must expose. |
| Ch. 3–4 (Responsibility, IR) | Normative in intent | Structural claims about how responsibilities and IR are formed. |
| Ch. 6–9 (Primitive → Validation) | Normative in intent | The execution and trust layers. |
| Ch. 1–2 | Informative | Motivation and positioning. Argue with these freely. |

The **glossary and the primitive specification are deliberately licensed more permissively than the surrounding prose** ([see licensing](#licensing)). We want the vocabulary and the primitive contract reused as widely as possible, including commercially. The argument can be debated; the vocabulary should spread.

## Status

**v0.1 — Draft.** Ch. 1–9 complete. Ch. 10–20 (reasoning compiler, runtime, domain specialization) not yet published.

This specification contains **known open issues**, recorded rather than hidden. See [Open issues](./primitives/index.md#open-issues). The most significant: the `category` field is defined with one value set in the TPS schema and exercised with a different, larger value set in the primitive examples. This is unresolved, and we would rather publish it as an open question than quietly pick one and pretend it was designed.

## Open by design

TAE describes how traffic agents *should* work. It does not require you to adopt any particular implementation.

The reference implementation lives in **WaySense** products. It is not required reading, and nothing here depends on it. If you implement TAE in a different stack, that is the specification working as intended — please [open an issue](https://github.com/WaySense-AI/traffic-agentic-engineering/issues) and tell us where the spec was unclear, because that is the most valuable signal this project can receive.

## Licensing

Dual, and deliberately so:

| Content | License | Rationale |
|---------|---------|-----------|
| Prose — chapters, argument, exposition | [CC BY-NC-SA 4.0](./LICENSE-CONTENT.md) | Reuse and translate freely; do not sell the text itself. |
| Glossary, TPS, primitive identifiers, diagrams | [CC BY 4.0](./LICENSE-SPEC.md) | Maximum reuse. Commercial use welcome. Attribution only. |

The asymmetry is the point. We are not trying to license revenue out of a specification. We are trying to make the vocabulary unavoidable.

## Citing

If TAE informs your work, please cite it — a machine-readable record is in [`CITATION.cff`](./CITATION.cff).

```bibtex
@techreport{tae2026,
  title  = {Traffic Agentic Engineering: An Open Specification of Traffic Agent Cognition},
  author = {Guo, Haifeng},
  year   = {2026},
  note   = {Volume I, Chapters 1--9. v0.1 draft.},
  url    = {https://github.com/WaySense-AI/traffic-agentic-engineering}
}
```

## Contributing

The highest-value contributions, in order:

1. **Report where the vocabulary is underspecified.** A term that two engineers would define differently is a bug in a specification.
2. **Propose primitives.** New `traffic.<domain>.<metric>` identifiers with a complete TPS record.
3. **Test the claims.** TAE makes falsifiable assertions — e.g. that reasoning expressed in primitives is reproducible across operators. Disproving one is a contribution.
4. **Translate.** See [`zh-CN/`](./zh-CN/index.md) for the Chinese edition.

---

*Traffic engineering is not a knowledge problem. It is a compilation problem.*
