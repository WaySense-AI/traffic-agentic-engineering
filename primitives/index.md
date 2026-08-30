---
layout: default
title: "Traffic Primitive Specification"
nav_order: 40
---

# Traffic Primitive Specification (TPS)

**Status: normative.** The semantic contract every engineering primitive must expose.

Together with the [glossary](../glossary/index.md), this is the part of TAE designed to be reused. It is licensed [CC BY 4.0](./LICENSE.md) — commercial use welcome, attribution required.

---

## The contract

An **engineering primitive** is a stable engineering concept. An **operator** is one computational way to realize it.

```
traffic.queue.length                    ← the primitive (stable)
    ├── CTM                             ← operators (swappable)
    ├── SUMO / VISSIM
    ├── Video analytics
    ├── Loop detector statistics
    ├── Radar
    └── Neural traffic foundation model
```

Six algorithms. Six accuracy profiles. Six cost structures. **One engineering meaning.**

This is not abstraction for its own sake. It buys two concrete properties:

1. **Technology independence.** When a better operator appears, the primitive is untouched. Only `default_operator` changes.
2. **Reproducibility.** A reasoning trace expressed in primitives can be re-executed against different operators and the results compared — which is the precondition for audit.

## Naming convention

```
traffic.<domain>.<metric>
```

Identifiers use **engineering terminology, never implementation names.**

| Correct | Incorrect | Why |
|---------|-----------|-----|
| `traffic.queue.length` | `CTMQueueCalculator` | CTM is one operator, not the concept |
| `traffic.delay.hcm` | `HCMDelayFunction` | Function name leaks implementation language |
| `traffic.saturation.degree` | `ComputeVCRatio` | Imperative verb describes code, not meaning |

The test: would the identifier still be correct if the field abandoned this algorithm entirely? If not, it names an implementation.

## TPS schema

Every primitive must expose the following. Note that TPS describes **reasoning interfaces**, not function signatures.

```
┌─────────────────────────────────────────────────────────────┐
│                  TRAFFIC PRIMITIVE SPECIFICATION             │
│                                                              │
│  Primitive Identity                                         │
│  ├── id: Globally unique identifier                         │
│  ├── name: Human-readable name                             │
│  ├── version: Semantic version number                      │
│  └── category: Classification (Observation / Analysis /      │
│      Evaluation / Decision)                                 │
│                                                              │
│  Semantics                                                  │
│  ├── meaning: What engineering concept does this represent? │
│  ├── role: Observation / Diagnostic / Evaluative / Normative│
│  ├── dimension: Congestion / Safety / Efficiency / ...      │
│  └── reasoning_contribution: How does this support inference?│
│                                                              │
│  Interface                                                  │
│  ├── inputs: Required engineering entities                  │
│  ├── outputs: Produced engineering quantities               │
│  └── evidence: What engineering conclusions can be drawn?   │
│                                                              │
│  Constraints                                                │
│  ├── preconditions: What must be true before execution?      │
│  ├── applicability: When is this primitive valid?           │
│  └── limitations: Known failure modes                       │
│                                                              │
│  Execution                                                  │
│  ├── operators: Available computational implementations     │
│  ├── default_operator: Recommended default                  │
│  └── cost_profile: Computational resource requirements      │
│                                                              │
│  Quality                                                    │
│  ├── accuracy: Expected error bounds                        │
│  ├── confidence: Output uncertainty characterization        │
│  ├── latency: Execution time range                          │
│  └── interpretability: Can results be explained?            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

The planner asks: *"Which primitive can establish the evidence I need, given my constraints and quality requirements?"* — not *"Which API should I call?"*

## Worked example: `traffic.queue.length`

**Semantics**

| Field | Value |
|-------|-------|
| **meaning** | Estimate the physical queue length (meters or vehicles) of a traffic movement during a specified period |
| **role** | Observation Primitive |
| **dimension** | Congestion / Spatial |
| **reasoning_contribution** | Enables spillback detection, storage utilization assessment, and congestion severity quantification |

Nothing in the semantics mentions CTM, SUMO, loop detectors, or any implementation. **Semantics are technology-independent by construction.**

**Interface** — inputs and outputs are *engineering entities*, not software objects:

| Input | Type | Description |
|-------|------|-------------|
| `LaneState` | Structured | Lane configuration, count, geometry |
| `SignalTiming` | Structured | Active plan: cycle, splits, offsets |
| `ArrivalFlow` | Time series | Vehicle arrival pattern |
| `TurningRatio` | Distribution | Through/left/right percentages |

| Output | Type | Unit |
|--------|------|------|
| `QueueLength` | Scalar | meters or vehicles |
| `QueueGrowthRate` | Scalar | m/min or veh/h |
| `StorageUtilization` | Ratio | queue_length / storage_length |

## Primitive catalog

The identifiers below appear in Volume I. This catalog is **illustrative, not complete** — see [Open issue #2](#open-issue-2-catalog-completeness).

| Identifier | Meaning | Category (as used) |
|------------|---------|--------------------|
| `traffic.queue.length` | Physical queue length of a traffic stream | Observation |
| `traffic.delay.hcm` | Control delay per HCM methodology | Evaluation |
| `traffic.delay.control` | Control delay (operator-independent form) | Evaluation |
| `traffic.capacity.lane` | Lane-level capacity estimate | Analysis |
| `traffic.signal.split` | Green split allocation per movement | Parameter |
| `traffic.offset.greenband` | Progression bandwidth / offset | Coordination |
| `traffic.storage.available` | Effective storage length vs. queue | Geometric |
| `traffic.geometry.storage` | Storage geometry of a lane group | Geometric |
| `traffic.saturation.degree` | Volume-to-capacity (v/c) ratio | Diagnostic |
| `traffic.los.grade` | Level of Service classification (A–F) | Evaluation |
| `traffic.los.assessment` | LOS assessment for a movement or approach | Evaluation |
| `traffic.crash.conflict` | Conflict point analysis outcome | Safety |
| `traffic.emission.factor` | Per-vehicle emission rate | Environmental |
| `traffic.weather.condition` | Observed weather state | Observation |
| `traffic.cycle.webster` | Cycle length per Webster's method | Analysis |

## Why primitives make results traceable

The value of the primitive layer is easiest to see as a contrast (Ch. 8):

> **Untraceable**
> "The eastbound approach experiences severe queuing during PM peak."

> **Traceable**
> "Queue length on the eastbound approach exceeded storage capacity (160m vs. 120m lane group length) during 17:00–18:00, as measured by `traffic.queue.length` primitive [Q_EB_PMpeak = 168m, σ = 12m, n = 22 observations] and confirmed by `traffic.geometry.storage` primitive [storage = 120m]."

Both sentences describe the same finding. Only one can be reviewed, challenged, or signed. The primitive layer is what converts a plausible statement into a defensible one — and in traffic engineering, **defensibility is the product**.

---

# Open issues

Published rather than resolved. A specification that hides its own disagreements is marketing.

## Open issue #1: `category` is defined with one value set and used with another

The TPS schema defines `category` as a four-value functional classification:

```
category: Classification (Observation / Analysis / Evaluation / Decision)
```

But the primitive identity table (Ch. 6.12) and the catalog above use a different, larger set of values — including `Parameter`, `Coordination`, `Geometric`, `Diagnostic`, `Safety`, and `Environmental`.

Compounding this, `role` is defined as a *separate* axis with four values (`Observation / Diagnostic / Evaluative / Normative`), which overlaps with both candidate `category` sets.

Three resolutions are available:

| Option | Reading | Consequence |
|--------|---------|-------------|
| **A** | `category` is the 4-value functional set; the catalog's values belong in a new `domain` field | Clean orthogonal axes; requires re-tagging all catalog entries |
| **B** | `category` is an open domain tag set; the schema box is out of date | Simpler, but `category` and `role` become ambiguous |
| **C** | Two axes: `category` (4-value functional) × `domain` (open) | Most expressive; most tooling burden |

No decision has been made. Option C is the current lean, on the grounds that `traffic.crash.conflict` is genuinely both a `Safety`-domain and an `Evaluation`-category primitive, and collapsing those loses information.

**Comments welcome** — this is exactly the kind of definitional question where outside use settles the answer faster than internal debate.

## Open issue #2: Catalog completeness

Volume I exercises 15 primitives. A production traffic agent needs on the order of 50 across the six engineering domains.

The naming convention is designed to permit extension without central coordination: any team may mint `traffic.<domain>.<metric>` identifiers. The open question is governance — whether a registry is needed to prevent collisions and near-duplicates (`traffic.delay.hcm` vs. `traffic.delay.control` already sit uncomfortably close).

Proposals for new primitives, with a complete TPS record, are the most valuable contribution this repository can receive.

---

**License**: [CC BY 4.0](./LICENSE.md)
