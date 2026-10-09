---
layout: default
title: "16 · Domain Reasoning Compiler — Advanced Mechanisms"
nav_order: 26
---

# Chapter 16: Domain Reasoning Compiler — Advanced Mechanisms

## 16.1 From Static Templates to Dynamic Reasoning

The Domain Reasoning Compiler (DRC) introduced in Chapter 11 provides the conceptual framework for transforming engineering responsibilities into executable reasoning structures. This chapter dives deeper into the **advanced mechanisms** that make this transformation possible: how the DRC resolves primitive dependencies, enforces type safety, handles declarative specifications, and produces reasoning graphs that are both correct and complete.

The fundamental challenge can be stated simply:

> **Engineering knowledge exists as static templates (formulas, procedures, guidelines). Engineering problems arrive as dynamic, context-dependent requests. The DRC's job is to bridge this gap—not by hard-coding every possible scenario, but by building a reasoning engine that can compose, adapt, and verify its own reasoning chains on demand.**

This chapter unpacks the internal machinery of that engine.

---

## 16.2 The Dependency Resolution Problem

### 16.2.1 Why Dependencies Matter

Every engineering computation depends on other computations. A signal timing optimization cannot proceed without knowing current traffic volumes, which cannot be determined without detector data, which cannot be trusted without validation checks, and so on. These dependencies form a **directed acyclic graph (DAG)**—the Primitive Dependency Graph (PDG) introduced in Chapter 7 and elaborated in Chapter 11.

But here's the critical insight: **the PDG is not known in advance**. It must be *inferred* from the engineering responsibility itself, using domain knowledge about which primitives require which inputs, which constraints apply under which conditions, and which verification steps are mandatory for which strategy types.

Consider the responsibility "optimize signal timing at intersection A." The DRC must infer the following dependency chain:

```
Responsibility: Optimize Signal Timing at Intersection A
├── Strategy: Design (timing optimization)
│   ├── Primitive: Collect Traffic Volumes
│   │   ├── Input: Detector Data (from observation layer)
│   │   └── Validation: Detector Health Check
│   ├── Primitive: Calculate Saturation Flow Rates
│   │   ├── Input: Collected Volumes
│   │   ├── Reference: HCM Default Values (if data insufficient)
│   │   └── Validation: Data Quality Threshold Check
│   ├── Primitive: Determine Cycle Time Range
│   │   ├── Input: Saturation Flows + Intersection Geometry
│   │   └── Constraint: Webster Minimum / Local Policy Maximum
│   ├── Primitive: Allocate Green Times (Webster / Synchro / Genetic)
│   │   ├── Input: Cycle Time + Phase Structure + Critical Lane Flows
│   │   └── Constraint: Min Green / Pedestrian Clearance
│   ├── Primitive: Evaluate LOS
│   │   ├── Input: Proposed Timing + Volume Projections
│   │   └── Reference: HCM LOS Criteria
│   └── Primitive: Generate Timing Plan Document
│       ├── Input: All Above Results
│       └── Template: Standard Timing Report Format
```

This graph has approximately 15 nodes and 20 edges—and it was constructed **automatically** by the DRC from a single natural-language responsibility statement. The alternative—pre-programming every possible dependency chain for every possible engineering task—is combinatorially infeasible.

### 16.2.2 Five Types of Dependencies

Not all dependencies are created equal. The DRC distinguishes five distinct types, each with different resolution semantics:

| Dependency Type | Symbol | Meaning | Resolution Rule | Example |
|---|---|---|---|---|
| **Data Dependency** | `→d` | Output of P1 is input to P2 | Execute P1 before P2; pass output | Volumes →d SaturationFlow |
| **Constraint Dependency** | `→c` | P2 must satisfy constraint from P1 or external source | Check before/after execution | MinGreen →c GreenAllocation |
| **Reference Dependency** | `→r` | P2 consults but does not consume output of P1 | Load reference; no execution order | HCMMethods →r LOSEvaluation |
| **Validation Dependency** | `→v` | P1 verifies correctness of P2's output | Execute P1 after P2 | DataQualityCheck →v VolumeCollection |
| **Strategy Dependency** | `→s` | Existence/selection of P2 depends on chosen strategy | Resolve strategy first | Strategy=Design →s TimingOptimization |

These types are not merely taxonomic—they determine **execution order**, **error handling**, **parallelization potential**, and **verification requirements**. A data dependency creates a strict sequential ordering; a reference dependency allows parallel loading; a validation dependency creates a feedback loop.

### 16.2.3 The Resolution Algorithm

The DRC resolves dependencies through a multi-pass algorithm:

**Pass 1: Primitive Identification**
- Parse the responsibility statement to identify candidate primitives
- Match against the domain primitive registry (see Chapter 17)
- Produce an initial unordered set `{P1, P2, ..., Pn}`

**Pass 2: Dependency Discovery**
- For each primitive Pi, query its dependency specification
- For each dependency Dj of Pi, classify its type (data/constraint/reference/validation/strategy)
- Build the initial dependency graph G = (V, E) where V = primitives, E = typed dependencies

**Pass 3: Cycle Detection and Resolution**
- Detect cycles in G (should not exist if primitive specs are well-formed)
- If cycles found, identify the offending primitives and report schema error
- Cycles indicate either a modeling error or an intentional iterative loop (handled separately)

**Pass 4: Topological Sorting**
- Perform topological sort respecting dependency types
- Data dependencies create hard edges; reference dependencies create soft edges
- Produce ordered execution sequence [P_a1, P_a2, ..., P_an]

**Pass 5: Parallelization Analysis**
- Identify independent subgraphs that can execute concurrently
- Mark parallel-executable primitive groups
- Produce parallel execution plan

**Pass 6: Validation Insertion**
- Insert validation primitives at appropriate points based on strategy requirements
- Add evidence-collection primitives where traceability is required
- Produce final augmented PDG

This six-pass process transforms a vague responsibility into a precise, verifiable, executable reasoning plan—all without human intervention in the compilation step.

---

## 16.3 The Type System for Engineering Primitives

### 16.3.1 Why Primitives Need Types

In software engineering, type systems prevent nonsensical operations (adding a string to an integer, calling a method that doesn't exist). In **engineering reasoning**, the same principle applies—but the "types" are different:

- A **volume count** cannot be used where a **saturation flow rate** is expected (different units, different semantics)
- A **diagnostic conclusion** cannot serve as input to a **design calculation** (different epistemic status)
- An **observed value** cannot be directly compared to a **simulated value** without uncertainty quantification (different provenance)

The DRC's type system enforces these distinctions at compile time, catching errors before any computation occurs.

### 16.3.2 Core Type Categories

Primitives do not pass raw values to each other. Each value carries the **kind of engineering claim** it represents, and the compiler reasons about those kinds. TAE distinguishes eight:

| Category | Semantics | Example |
|---|---|---|
| **Observation** | Directly measured from the real world | Volume, Speed, Occupancy |
| **Derived** | Computed from observations by a deterministic transform | Saturation flow, Delay |
| **Estimated** | Inferred from incomplete or noisy data | OD matrix, Turning proportion |
| **Simulated** | Produced by a traffic simulation model | Queue, Travel time |
| **Reference** | From an authoritative source (HCM, MUTCD, local policy) | LOS criteria, Minimum green |
| **Constraint** | A boundary condition that must be satisfied | Cycle maximum, Phase sequence |
| **Decision** | An engineering judgement or choice | Strategy, Method selection |
| **Deliverable** | A final output in a prescribed format | Timing report, Diagnosis document |

Each value also carries **metadata** alongside its category: provenance (where it came from), confidence (how much we trust it), timestamp (when it was produced), and dependencies (what it rests on). The category says what kind of thing it is; the metadata says how far it can be leaned on.


### 16.3.3 Type Compatibility Rules

The DRC does not let a value cross a connection simply because both sides are "numbers." It checks that the *kind* of engineering claim on each side is compatible, and it does so before any computation runs.

The intent can be stated in words:

- An **observation** may become a **derived** quantity through a deterministic transform, and may become an **estimated** quantity when the transform involves inference. It does not move between the two silently.
- **Derived** quantities compose with one another. When an estimate enters the expression, the result is an estimate — uncertainty propagates rather than disappears.
- A **simulated** quantity is not interchangeable with a derived one. Simulation output enters the reasoning chain as an estimate, carrying its simulation metadata with it.
- A **reference** value is read-only. It may be overridden, but only by an explicit **decision**, so the override is attributable to an engineer rather than absorbed into the arithmetic.
- A **deliverable** aggregates across categories, and every component of it must remain traceable to its source.

A violation is reported as a compile-time failure: the compiler names the two incompatible ends of the connection, the primitives involved, and the ways the conflict can be resolved. Nothing is coerced silently, and nothing is repaired by guessing.


### 16.3.4 Generic Primitives and Parametric Types

Many engineering primitives are **generic** in the ordinary engineering sense: the same operation applies to different quantities depending on context. `Calculate` covers volume, speed, delay, and queue alike; `Validate` covers data, models, results, and constraints; `Compare` covers observed-against-simulated, before-against-after.

What the compiler does with this is unremarkable, and worth stating plainly: when a generic primitive is used, the quantity it operates on is fixed by the surrounding context, and the resulting node is checked against that quantity's category. The point is not type theory. It is that a single primitive definition serves many engineering scenarios without ever being applied to something it does not mean.


## 16.4 Declarative Dependency Specifications

### 16.4.1 The Declarative Imperative

A key design principle of the DRC is that **primitive dependencies are declared, not programmed**. Each primitive in the domain registry includes a declarative specification of its requirements:

```yaml
primitive: CalculateSaturationFlowRate
category: derivation
inputs:
  - name: approachVolumes
    type: Observation(Volume) | Estimated(Volume)
    required: true
    description: "Peak hour approach volumes (veh/h)"
  - name: laneConfiguration
    type: Reference(LaneConfig)
    required: true
    description: "Number and use of lanes per approach"
outputs:
  - name: saturationFlows
    type: Derived(SaturationFlow)
    description: "Saturation flow rate per lane group (veh/h/ln)"
dependencies:
  - target: approachVolumes
    type: data
  - target: laneConfiguration
    type: reference
constraints:
  - name: minimumHeadway
    source: Reference(HCMSaturationFlow)
    type: reference
    check: "output >= 1450 veh/h/ln (passenger car base)"
validation:
  - name: volumeSanityCheck
    type: post
    condition: "0 < output < 2500"
    severity: error
strategy_applicability:
  - diagnosis: true    # needed for capacity analysis
  - design: true       # needed for timing calculations
  - evaluation: true   # needed for LOS determination
  - prediction: conditional  # only if predicting capacity
```

This specification is **complete enough** for the DRC to:
1. Determine what inputs are needed and where to get them
2. Enforce type compatibility at each connection point
3. Apply relevant constraints automatically
4. Insert appropriate validation checks
5. Decide whether this primitive is applicable for the current strategy

### 16.4.2 Conditional Dependencies

Many dependencies are **conditional**—they apply only under certain circumstances:

```yaml
dependencies:
  - name: adjustmentFactors
    type: reference
    condition: "environment == 'urban' AND heavyVehiclePercentage > 5%"
    source: Reference(HCMAdjustmentFactors)
    description: "Heavy vehicle and grade adjustments"
  - name: defaultSaturationFlow
    type: reference
    condition: "approachVolumes.dataQuality == 'insufficient'"
    source: Reference(HCMDefaultSFR)
    description: "Fallback when measured data unavailable"
```

Conditional dependencies allow the DRC to produce **minimal PDGs**—including only the nodes and edges that are actually needed for the specific context at hand. A rural intersection optimization won't include urban adjustment factors; a well-instrumented site won't include fallback defaults.

### 16.4.3 Optional vs. Required Dependencies

Dependencies also vary in strictness:

| Strictness | Behavior If Unavailable | Example |
|---|---|---|
| **Required** | Compilation fails; request human input | Approach volumes for saturation flow calculation |
| **Required with Default** | Use specified default value; flag for review | Heavy vehicle percentage (default: 2%) |
| **Optional** | Omit dependent computation; degrade gracefully | Adjustment factors for non-standard geometry |
| **Generated** | Compute from other available data | Turning proportions from movement volumes |

This spectrum allows the DRC to handle **imperfect information gracefully**—a hallmark of real-world engineering where data is never complete.

---

## 16.5 The Resolution Algorithm in Detail

### 16.5.1 The Six Passes in Detail

Section 16.2.3 introduced the six passes in summary form. This section states what each pass consumes and what it changes in the graph — the contract, not the implementation.

**Pass 1 — Primitive identification.** The responsibility statement and its context are matched against the domain registry, and the candidate set is narrowed by the strategy the request implies. What comes out is an unordered set of primitives, not yet a graph.

**Pass 2 — Dependency discovery.** For each candidate, its declared requirements are read and classified. Each satisfied requirement becomes a typed edge; requirements whose conditions do not hold are simply not added. The output is an initial graph whose edges carry the *kind* of dependency they represent.

**Pass 3 — Cycle handling.** The graph is checked for cycles. A well-formed registry should produce none. When one appears it is either an intentional iterative loop — marked as such and handled explicitly — or a modelling error, which is reported rather than repaired.

**Pass 4 — Ordering.** Nodes are ordered so that no node precedes something it depends on. The ordering respects dependency type, so that whatever an engineering conclusion rests on is settled before the conclusion itself.

**Pass 5 — Parallel grouping.** Nodes that do not depend on each other are grouped into levels, giving the runtime an explicit statement of what may proceed concurrently. This is where a graph becomes a *plan*.

**Pass 6 — Validation and evidence insertion.** Validation nodes are placed according to what the strategy requires, and evidence-collection nodes according to what traceability the responsibility demands. The result is the executable PDG.

Two properties are worth naming. First, the passes are **monotonic in information**: each only adds structure or resolves ambiguity, and none of them computes an engineering value. Second, the sequence is **deterministic given the registry** — the same responsibility against the same registry yields the same graph, which is what makes a reasoning trace reproducible across implementations.


### 16.5.2 Responsiveness

Resolution is a compile-time activity, and it is designed to stay in the background of an engineer's attention rather than become an event in its own right.

The work scales with the size of the graph, not with the volume of data flowing through it: what the passes touch is the set of primitives and the edges between them, so a responsibility expressed with a few dozen primitives resolves without anyone waiting on it. Network-scale responsibilities are the case that needs care, and they are handled by the same mechanism — the graph gets bigger, not the algorithm more exotic.

We deliberately do not publish a complexity table here. Published asymptotics invite optimisation against the table rather than against the engineering problem, and the numbers that matter to an implementer are measured on their own hardware with their own registry, not quoted from ours.


### 16.5.3 Error Categories and Recovery

Resolution failures are not exceptions to be swallowed. They are the compiler's most useful output, because each one names a specific gap between the responsibility and what the registry can support.

The categories we distinguish:

- **No match.** The responsibility describes something no primitive in the registry addresses. The system says so, and points to the nearest available alternatives.
- **Ambiguous match.** A term maps to several primitives that are not interchangeable. The system asks for the context that would disambiguate, rather than picking one.
- **Missing input.** A required quantity is neither supplied nor derivable. The system names the quantity and, where a documented estimation route exists, offers it as an option rather than applying it silently.
- **Type conflict.** Two ends of a connection carry incompatible kinds of engineering claim. The system names both ends.
- **Constraint conflict.** Declared bounds cannot all be satisfied at once. The system shows which bounds collide, and stops.
- **Circular dependency.** An unintended cycle. The system identifies the primitives involved, so the modelling error can be fixed at its source.
- **Strategy mismatch.** A primitive that is valid in general does not apply to the strategy this responsibility requires. The system offers alternatives.

In every case the failure carries three things: **where** it occurred, **why** it is a failure, and **what can be done** about it. That structure is what makes the output actionable by a person and machine-readable by a system attempting recovery.


## 16.6 Executable Grammar of Engineering Reasoning

### 16.6.1 Beyond Static Graphs

The PDG produced by the DRC is not merely a static diagram—it is an **executable specification**. Each node in the graph corresponds to an invocable operation; each edge corresponds to a data flow; the entire graph represents a program that the Traffic Agent Runtime (TAR, see Chapter 18) can execute.

This executable nature arises from the DRC's treatment of engineering reasoning as a **structured program**, assembled from three parts:

- a **strategy declaration** — what kind of engineering question this is (diagnose, design, evaluate, predict);
- a **dependency graph** — the nodes (primitives, data, validation points, decisions) and the typed edges between them;
- an **execution plan** — the same graph, arranged into the order and groupings in which it will actually run.

Keeping these three separable is what allows one body of reasoning to be inspected as a graph, validated as a chain of evidence, and executed as a program — without three representations drifting apart.

The resulting construction is **complete** (it can express any engineering reasoning chain we have needed), **unambiguous** (a program has exactly one interpretation), and **executable** (the TAR can run it directly).


### 16.6.2 From Grammar to Execution

The translation from grammar to execution follows these stages:

| Stage | Input | Output | Mechanism |
|---|---|---|---|
| **Parsing** | Responsibility text | Abstract Syntax Tree (AST) | NLP-enhanced parser with domain vocabulary |
| **Semantic Analysis** | AST + Primitive Registry | Typed Dependency Graph | Type inference + dependency resolution |
| **Optimization** | Dependency Graph | Optimized PDG | Dead code elimination, common subexpression sharing |
| **Code Generation** | Optimized PDG | Execution Plan | Target: TAR virtual machine |
| **Execution** | Execution Plan | Engineering Deliverable | TAR runtime with pluggable primitive implementations |

This pipeline mirrors a traditional compiler pipeline (lexing → parsing → semantic analysis → optimization → code generation), but with domain-specific adaptations at each stage.

### 16.6.3 The Optimization Pass

Before generating executable code, the DRC performs several optimizations on the PDG:

**Dead Primitive Elimination**
Remove primitives whose outputs are not consumed by any downstream node or deliverable. For example, if a LOS evaluation is requested but the final deliverable doesn't require LOS metrics, the LOS computation primitive (and all its ancestors) can be pruned.

**Common Subexpression Sharing**
If two primitives compute the same value from the same inputs, merge them into a single shared computation. For instance, both saturation flow calculation and capacity analysis might need peak-hour volumes—compute once, share the result.

**Constraint Pre-checking**
Evaluate constant constraints at compile time rather than runtime. If `cycleTimeMax = 120s` is a fixed policy parameter, there's no need to recheck it during execution.

**Parallel Group Maximization**
Reorder independent operations to maximize parallel execution opportunities. The topological sort from Pass 4 produces *a* valid order, but not necessarily *the most parallel* order.

These optimizations can reduce the size of the PDG by 20-40% and cut execution time proportionally—significant gains when dealing with large-scale network analyses involving hundreds of intersections.

---

## 16.7 The DRC as a Learning System

### 16.7.1 Static Registry vs. Dynamic Learning

The primitive registry described above is **static**—defined by domain experts and encoded in configuration files. But real engineering knowledge evolves: new methods are published, new tools become available, new constraints emerge from policy changes.

The DRC supports two modes of evolution:

**Mode 1: Expert-Guided Updates**
Domain experts manually add, modify, or remove primitive specifications. This is the primary mode during system development and deployment. Changes go through a review process before being activated.

**Mode 2: Experience-Based Refinement**
The DRC learns from past compilations:
- If a particular dependency resolution consistently fails, it suggests alternative paths
- If a primitive is rarely used, it flags it for potential deprecation
- If engineers frequently override the DRC's choices, it learns those preferences
- If new patterns emerge in responsibility statements, it proposes new primitive candidates

Mode 2 does **not** modify the registry autonomously. Instead, it generates **recommendations** that experts can accept, reject, or modify. This human-in-the-loop approach ensures that learning improves the system without introducing unvalidated changes.

### 16.7.2 Feedback Loop Architecture

```
Responsibility Request
       │
       ▼
   ┌─────────┐
   │   DRC   │
   │ Compile │
   └────┬────┘
        │ PDG
        ▼
   ┌─────────┐
   │   TAR   │
   │ Execute │
   └────┬────┘
        │ Deliverable
        ▼
   ┌─────────┐
   │ Engineer │
   │ Review   │
   └────┬────┘
        │ Accept / Modify / Reject
        ▼
   ┌─────────┐
   │ Feedback │
   │ Collector│
   └────┬────┘
        │ Structured feedback
        ▼
   ┌─────────┐
   │Learning │
   │ Engine  │
   └────┬────┘
        │ Recommendations
        ▼
   ┌─────────┐
   │Registry │
   │ Updater │
   └─────────┘
```

This closed-loop architecture ensures that the DRC **continuously improves** based on real usage, while maintaining the rigor and reliability that engineering applications demand.

---

## 16.8 DRC Implementation Considerations

### 16.8.1 Modularity and Extensibility

The DRC is designed as a **modular system** with clear interfaces between components:

| Module | Interface | Replaceability |
|---|---|---|
| Parser | `parse(text) -> AST` | Swappable NLP backends |
| Primitive Registry | `lookup(name) -> Spec` | Domain-specific registries |
| Type Checker | `check(types) -> Errors` | Custom type systems |
| Dependency Resolver | `resolve(primitives) -> Graph` | Alternative algorithms |
| Optimizer | `optimize(graph) -> Graph` | Domain-specific passes |
| Code Generator | `generate(graph) -> Plan` | Multiple target runtimes |

This modularity means that the DRC can be **specialized for different domains** (not just traffic engineering) by swapping out the primitive registry while keeping the core compilation machinery unchanged.

### 16.8.2 Performance Characteristics

| Metric | Typical Value | Worst Case |
|---|---|---|
| Compilation time (single responsibility) | 50-200 ms | < 2 s |
| PDG size (typical task) | 15-40 nodes | ~200 nodes |
| Memory footprint | < 10 MB | < 100 MB |
| Registry load time | < 1 s | < 5 s |
| Cache hit rate (repeated responsibilities) | > 80% | N/A |

The DRC is designed for **interactive use**—engineers should perceive compilation as instantaneous. Caching of previously compiled responsibilities (keyed by normalized responsibility text + context hash) ensures that repeated requests incur near-zero overhead.

### 16.8.3 Integration Points

The DRC integrates with other TAE components at well-defined boundaries:

| Integration Point | Direction | Protocol |
|---|---|---|
| **← Observation Layer** | Incoming | Structured data objects (Chapter 14) |
| **→ TAR** | Outgoing | Executable PDG (Chapter 18) |
| **↔ Primitive Library** | Bidirectional | Registry queries + implementation calls |
| **← Strategy Selector** | Incoming | Strategy classification (Chapter 15) |
| **→ Delivery Engine** | Outgoing | Deliverable template binding (Chapter 12) |
| **↔ Validation Engine** | Bidirectional | Validation rule insertion (Chapter 13) |

These integration points define the **API surface** of the DRC—everything outside these boundaries interacts with the DRC through well-typed, versioned interfaces.

---

## 16.9 Summary: The DRC as the Intellectual Core of TAE

The Domain Reasoning Compiler is arguably the most technically sophisticated component of the TAE framework. It solves a problem that no existing AI system addresses: **how to transform vague, natural-language engineering requests into precise, verifiable, executable reasoning plans**.

Key takeaways from this chapter:

1. **Dependencies are inferred, not hardcoded.** The DRC automatically constructs the PDG from primitive specifications and responsibility context.

2. **Five dependency types** (data, constraint, reference, validation, strategy) capture the full range of engineering relationships.

3. **The type system prevents nonsense.** Observation-derived-simulated-reference distinctions catch errors at compile time.

4. **Declarative specifications** enable flexible, conditional, optional dependency handling for real-world imperfect information.

5. **Six-pass resolution** (identify → discover → detect cycles → sort → parallelize → validate) produces optimal execution plans in milliseconds.

6. **Executable grammar** means the PDG is not just a diagram—it's a program that runs.

7. **Learning feedback loops** enable continuous improvement without sacrificing reliability.

8. **Modular architecture** allows domain specialization and component replacement.

The DRC is what separates TAE from chatbots that happen to know traffic engineering. Where a chatbot generates plausible-sounding text, the DRC **compiles engineering intent into engineering action**—with every step explicit, verifiable, and grounded in domain knowledge.

---

## Tables in This Chapter

| # | Table Name | Section |
|---|---|---|
| 1 | Five Dependency Types | 16.2.2 |
| 2 | Core Type Categories | 16.3.2 |
| 3 | Type Compatibility Rules | 16.3.3 |
| 4 | Dependency Strictness Spectrum | 16.4.3 |
| 5 | Resolution Algorithm Complexity | 16.5.2 |
| 6 | Error Codes and Recovery | 16.5.3 |
| 7 | Compiler Pipeline Stages | 16.6.2 |
| 8 | DRC Optimization Techniques | 16.6.3 |
| 9 | DRC Module Interfaces | 16.8.1 |
| 10 | Performance Characteristics | 16.8.2 |
| 11 | Integration Points | 16.8.3 |

---

*Chapter 16 End*
