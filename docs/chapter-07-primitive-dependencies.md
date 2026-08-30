---
layout: default
title: "7 · Primitive Dependencies — The Grammar of Engineering Reasoning"
nav_order: 17
---

# Chapter 7: Primitive Dependencies — The Grammar of Engineering Reasoning

## 7.1 A Graph Is Not the Innovation

Graph has become one of the most popular abstractions in modern AI systems. Knowledge Graphs connect entities. Scene Graphs connect visual objects. Computation Graphs connect mathematical operations. Workflow Graphs connect executable tasks. LangGraph connects agent states.

Despite their differences, these graphs share one common characteristic: **the graph itself is not the source of intelligence**. The intelligence lies in the semantics of the edges. Without meaningful dependencies, a graph is merely a collection of disconnected nodes — a data structure without meaning, a skeleton without flesh.

This observation is not merely philosophical. It carries profound practical implications for how we design engineering intelligence systems. When a traffic engineer approaches a problem such as signal timing optimization or congestion diagnosis, they do not begin by drawing boxes and arrows on a whiteboard. They begin by asking: *what depends on what?* What engineering concepts must be established before others can be evaluated? What evidence must be gathered before conclusions can be drawn?

Traffic Agentic Engineering therefore does not begin by asking: *How should primitives be connected?* Instead, it asks: **Why must two primitives be connected?** Only after this question is answered can a valid reasoning graph emerge. The graph is the *consequence* of engineering reasoning, not its *starting point*. This inversion — from graph-as-design to graph-as-consequence — is one of the fundamental distinctions between TAE and conventional workflow-based AI systems.

> **The Fundamental Inversion**: Workflow systems ask "what steps should be executed?" TAE asks "what knowledge depends on what?" The former produces execution plans; the latter produces reasoning structures. These are fundamentally different artifacts serving fundamentally different purposes.

Consider the analogy with natural language. A sentence's grammatical structure (its syntax tree) emerges from the semantic relationships between words, not from some pre-designed template. Similarly, a Primitive Dependency Graph emerges from the semantic relationships between engineering concepts. The grammar of engineering reasoning, like the grammar of language, is discovered rather than invented.

## 7.2 Dependency Is Engineering Knowledge

Suppose an engineer wishes to evaluate the Level of Service (LOS) at a signalized intersection. Can LOS be computed directly from raw sensor data? No. Before LOS can be established, several prerequisite engineering concepts must already be known and quantified:

**Arrival Flow → Capacity → Degree of Saturation → Control Delay → LOS**

Each arrow in this chain represents an irreversible dependency: you cannot compute degree of saturation without knowing both arrival flow and capacity; you cannot compute control delay without knowing degree of saturation; and you cannot assess LOS without knowing control delay. These relationships are not implementation details that can be rearranged at will. They are **engineering knowledge** — invariant truths about how traffic engineering concepts relate to one another.

The critical insight is that these dependencies survive across all possible algorithmic implementations. No matter whether delay is calculated by the HCM analytical formula, estimated from floating-car trajectory data, predicted by a neural operator, or approximated through simulation, **LOS still depends upon delay**. The dependency belongs to engineering semantics, rather than computational implementation. It is a property of the domain, not a property of any particular algorithm.

This algorithm-independence is what makes dependency graphs so powerful as an abstraction layer. When algorithms evolve — as they inevitably do — the dependency structure remains stable. Webster's method gave way to HCM, which is now giving way to data-driven and neural approaches. Yet the conceptual dependency chain from flow to capacity to saturation to delay to LOS has remained unchanged for over half a century. Traffic Agentic Engineering calls these relationships **Engineering Dependencies**, and elevates them to first-class citizens in the architecture of engineering intelligence.

| Dependency Type | Example Chain | Algorithm-Independent? | Stability |
|---|---|---|---|
| Semantic Definition | Flow → Capacity → Saturation | Yes | Decades |
| Evidence Requirement | Detector Data → Queue Length | Partially | High |
| Constraint Verification | Signal Plan → Minimum Green | Yes | Permanent |

## 7.3 Dependency Is Not Execution Order

A common and consequential misunderstanding is to interpret graph edges as prescribing execution order. This conflation of *dependency* with *scheduling* leads to architectures that are both rigid and incorrect. Consider the following example from intersection operations:

**Queue → Delay → LOS**

A workflow designer might read this as: "first execute the Queue computation, then execute Delay, then finally compute LOS." This interpretation is **incorrect**. The graph does not state that Queue must be executed before Delay. Instead, it states that **Delay cannot be established unless sufficient queue-related evidence exists**. The edge represents *semantic dependency*, not *runtime scheduling*.

The distinction matters profoundly for three reasons:

**First, independence enables parallelism.** If two primitives share no dependency relationship, they can be executed concurrently. Queue computations for different approach movements are independent of each other and can run in parallel. A workflow system that enforces sequential execution would waste this parallelism unnecessarily.

**Second, dependency does not imply immediacy.** The fact that Delay depends on Queue does not mean Queue must be computed fresh every time Delay is needed. Queue evidence may have been computed previously, cached, and remain valid. The dependency specifies a *knowledge requirement*, not a *computation trigger*.

**Third, execution remains the responsibility of the runtime scheduler.** The Primitive Dependency Graph only defines *what knowledge depends upon what other knowledge*. How and when to acquire that knowledge is a separate concern, delegated to the Domain Compiler and runtime scheduler introduced in subsequent chapters. This separation of concerns — between reasoning structure (the graph) and execution strategy (the scheduler) — is essential for building flexible, adaptive engineering intelligence systems.

> **Architectural Principle**: The Dependency Graph answers "what must be known before what?" The Runtime Scheduler answers "when and how should it be computed?" Confusing these questions produces systems that are either too rigid (workflow engines) or too chaotic (unstructured agent loops).

## 7.4 The Five Types of Engineering Dependency

Engineering dependencies are not homogeneous. Traffic Agentic Engineering classifies them into five distinct categories, each capturing a different mode of engineering relationship. Understanding these categories is essential for constructing correct reasoning graphs and for implementing effective dependency resolution algorithms.

### (1) Semantic Dependency

One engineering concept is defined in terms of another. The dependent concept has no independent meaning without its antecedent.

**Example**: Delay → LOS. Level of Service, as defined by HCM and adopted worldwide, is a classification scheme based explicitly on control delay. An LOS assessment has no meaning without an underlying delay value. The dependency is definitional and absolute.

Semantic dependencies form the backbone of engineering ontology. They establish the hierarchical structure of domain concepts and cannot be violated without redefining the concepts themselves. They are the most fundamental and immutable type of dependency.

### (2) Evidence Dependency

One engineering conclusion requires evidence produced by another primitive. The dependent primitive cannot produce reliable output without input from its antecedent.

**Example**: Capacity → Demand Overload. The conclusion that demand exceeds capacity (demand overload) cannot be reached without first establishing what the capacity actually is. Capacity provides the evidentiary baseline against which demand is compared.

Evidence dependencies differ from semantic dependencies in that they are empirical rather than definitional. They describe what information is needed to draw a conclusion, not what a concept means. They can sometimes be relaxed or approximated (e.g., using historical average capacity when real-time capacity estimation is unavailable), whereas semantic dependencies cannot.

### (3) Constraint Dependency

A primitive requires another primitive to verify engineering constraints. The constraint must be satisfied before the dependent primitive's output can be considered valid.

**Example**: Signal Timing → Minimum Green → Pedestrian Safety. A proposed signal timing plan depends on the verification that minimum green times satisfy pedestrian crossing requirements. The signal plan is constrained by pedestrian safety considerations, not vice versa.

Constraint dependencies introduce a directional quality to reasoning: they define which engineering concerns take precedence over others. They are essential for ensuring that engineering outputs satisfy regulatory, safety, and operational requirements.

### (4) Temporal Dependency

Engineering reasoning often spans time. Current conclusions depend on past observations, and future projections depend on current state.

**Example**: Queue(t) → Queue(t+1) → Queue Growth. The analysis of queue growth over time requires temporal observations at multiple time steps. Queue at time t+1 cannot be understood without reference to queue at time t. The dependency exists across the time dimension.

Temporal dependencies are particularly important in traffic engineering because traffic conditions are inherently dynamic. Peak hour buildup, spillback propagation, and congestion dissipation are all temporal phenomena that require time-series reasoning. The dependency graph must capture these temporal relationships explicitly.

### (5) Spatial Dependency

Traffic engineering is inherently spatial. Conditions at one location affect conditions at other locations through physical propagation effects.

**Example**: Upstream Queue → Downstream Spillback → Gridlock Risk. The queue forming at an upstream intersection can cause spillback into a downstream intersection, which in turn increases gridlock risk for the entire network. The dependency exists across space, rather than within a single intersection.

Spatial dependencies are what distinguish network-level traffic engineering from isolated intersection analysis. They enable reasoning about corridor-wide phenomena, network-wide congestion propagation, and regional traffic dynamics. Capturing spatial dependencies correctly is essential for any engineering intelligence system that operates beyond the single-intersection scope.

| Dependency Category | Direction | Violation Consequence | Example |
|---|---|---|---|
| Semantic | Definitional | Concept becomes meaningless | Delay → LOS |
| Evidence | Empirical | Conclusion unreliable | Capacity → Overload |
| Constraint | Regulatory | Output invalid/unsafe | Signal Time → Min Green |
| Temporal | Chronological | Dynamic analysis impossible | Queue(t) → Queue(t+1) |
| Spatial | Topological | Network reasoning fails | Upstream Q → Spillback |

## 7.5 Dependencies Grow the Graph

Unlike workflow systems where process graphs are manually designed by human engineers, Primitive Dependency Graphs in TAE **emerge automatically** through a process called *dependency expansion*. This automatic construction is not merely a convenience — it is a fundamental requirement for handling the complexity of real-world engineering problems.

Consider the responsibility: **Optimize Signal Timing** at an isolated intersection. The Reasoning Planner (introduced in Chapter 5) identifies that this responsibility requires the primitive `Signal Timing`. Immediately, the system queries the dependency specifications of `Signal Timing` and discovers its direct dependencies:

```
Signal Timing
├── Queue
├── Delay
├── Capacity
├── Offset
```

The planner then recursively expands each dependency. `Capacity` expands into:
```
Capacity
├── Arrival Flow
├── Lane Geometry
├── Saturation Flow Rate
├── Green Split
```

`Delay` expands into:
```
Delay
├── Queue
├── Signal Timing (cycle length, splits)
├── Arrival Pattern
```

The reasoning graph therefore grows layer by layer, like a tree expanding from its root. Each expansion step applies engineering knowledge encoded in primitive dependency declarations. The process terminates when all leaf nodes are primitives whose dependencies are fully satisfied by available evidence (e.g., raw detector data, geometric attributes, or cached computation results).

This recursive expansion process has several important properties:

**It is deterministic.** Given the same responsibility and the same dependency specifications, the same graph will always be produced. There is no randomness or heuristic search involved.

**It is complete.** The expansion process guarantees that all prerequisite knowledge is discovered. No dependency is overlooked because the process systematically explores the entire dependency tree.

**It is efficient.** Through memoization and duplicate elimination, the same primitive is never expanded twice. If `Queue` appears as a dependency of both `Signal Timing` and `Delay`, it is resolved once and shared.

**It is explainable.** The resulting graph documents exactly why each primitive is included — it is there because some other primitive depends on it, traceable back to the original responsibility.

Graph construction is thus an **engineering reasoning process**, rather than a workflow design exercise. The graph encodes the engineer's logic, made explicit and machine-executable.

## 7.6 Dependencies Form an Engineering Type System

This is perhaps the most important property of Primitive Dependency Graphs, and the one that distinguishes TAE most sharply from generic graph-based AI systems: **not every primitive can connect to every other primitive**. Connections must satisfy engineering semantics.

Consider the following two dependency chains:

- **Valid**: Queue → Delay (queue length is a necessary input for delay estimation)
- **Invalid**: Queue → Weather → Cycle Length (weather may correlate with traffic patterns, but there is no direct engineering dependency between weather and cycle length)

The first chain represents a legitimate engineering relationship recognized by traffic engineering theory. The second chain, while perhaps statistically correlational, does not represent a valid engineering dependency. Connecting these primitives would produce nonsense reasoning — the kind of error that plagues systems based purely on statistical association or unconstrained graph connectivity.

Traffic Agentic Engineering therefore introduces an **Engineering Type System** that governs how primitives may be connected. Every primitive exposes four type signatures:

| Type Signature | Description | Example (`traffic.queue.length`) |
|---|---|---|
| **Semantic Type** | What engineering concept does this primitive represent? | `QueueMeasurement` |
| **Input Type** | What evidence types does it consume? | `DetectorCounts`, `TimeInterval` |
| **Output Type** | What evidence type does it produce? | `QueueLengthEstimate` |
| **Evidence Type** | What is the epistemological status of its output? | `EmpiricalMeasurement` |

A dependency from primitive A to primitive B is **type-safe** only when B's output type is compatible with (or convertible to) one of A's required input types. If `traffic.delay.control` requires input of type `QueueLengthEstimate` and `traffic.queue.length` produces output of type `QueueLengthEstimate`, then the dependency is valid. If instead `traffic.weather.condition` produces output of type `WeatherObservation`, which is not convertible to `QueueLengthEstimate`, then the dependency is rejected by the type system.

Reasoning graphs therefore become **type-safe**. The type system prevents nonsensical connections, catches errors at graph-construction time (rather than at runtime when garbage output is produced), and serves as documentation of the engineering semantics embedded in the system. It is, in essence, the same role that type systems play in programming languages — except that the "programs" being type-checked are engineering reasoning processes.

> **Type Safety = Engineering Correctness**: Just as a compiler's type checker prevents a program from adding a string to an integer, TAE's engineering type system prevents a reasoning graph from feeding weather observations into a delay calculation. Both catch category errors before they can cause damage.

## 7.7 Dependencies Are Declarative

Engineering dependencies should never be hardcoded inside algorithms. Doing so would embed engineering knowledge into implementation code, where it becomes invisible, untestable, and unevolvable. Instead, dependencies are declared as **engineering knowledge** — first-class artifacts that exist independently of any algorithm or execution engine.

The declarative form of a primitive's dependencies might look like this:

```yaml
Primitive: traffic.delay.control
DependsOn:
  - traffic.queue.length        # Evidence dependency
  - traffic.signal.cycle       # Evidence dependency
  - traffic.signal.split       # Evidence dependency
Produces: delay_evidence         # Output type
EvidenceType: DerivedMeasurement # Epistemological status
---
Primitive: traffic.los.assessment
DependsOn:
  - traffic.delay.control      # Semantic dependency (definitional)
Produces: los_assessment
EvidenceType: Classification
```

Because dependencies are declarative, the compiler can perform powerful automated operations on reasoning graphs:

- **Analysis**: Detect unreachable primitives, orphaned subgraphs, and missing dependencies.
- **Optimization**: Eliminate redundant computations, merge duplicate paths, and reorder for efficiency.
- **Verification**: Prove that all dependencies are satisfied, all constraints are respected, and no circular references exist.
- **Evolution**: When engineering knowledge changes (e.g., a new dependency is discovered), update the declaration and let the compiler propagate the change automatically throughout all affected reasoning graphs.

This declarative approach mirrors the evolution of infrastructure-as-code in software engineering. Just as Terraform and Kubernetes manifests declare desired infrastructure state (leaving the how to the orchestrator), TAE dependency declarations declare desired reasoning structure (leaving the execution to the Domain Compiler). The benefit is the same: **separation of intent from mechanism**.

## 7.8 Dependency Resolution

Once dependencies become explicit and declarative, the compiler can resolve them automatically. This process — **dependency resolution** — is the mechanical procedure by which a complete Primitive Dependency Graph is constructed from an initial target primitive.

Given a target primitive (e.g., `traffic.los.assessment`), the resolution algorithm performs the following steps:

1. **Recursive Discovery**: Starting from the target, recursively discover all prerequisite primitives by following declared dependency links.
2. **Duplicate Elimination**: Remove duplicates — if the same primitive is reachable via multiple paths, keep only one instance.
3. **Circular Reference Detection**: Verify that the dependency graph contains no cycles. (Cycles indicate modeling errors — engineering dependencies should form a directed acyclic graph.)
4. **Semantic Compatibility Check**: Verify that all dependency edges satisfy the engineering type system. Reject invalid connections.
5. **Leaf Pruning**: Identify leaf nodes (primitives with no unresolved dependencies) that can be satisfied directly from available evidence sources.
6. **Graph Assembly**: Produce the complete, validated Primitive Dependency Graph.

This process resembles dependency resolution in modern package managers such as npm, pip, or cargo — except that **the objects being resolved are engineering concepts, rather than software libraries**. The resolver ensures that all prerequisites are available, versions are compatible, and conflicts are detected. But instead of downloading packages from a registry, the TAE resolver constructs a reasoning structure from engineering knowledge.

The output of dependency resolution is a complete, validated, type-safe Primitive Dependency Graph — ready to be passed to the Domain Compiler for transformation into an executable form.

| Resolution Step | Analog in Package Management | TAE Equivalent |
|---|---|---|
| Recursive Discovery | Transitive dependency scanning | Follow DependsOn links |
| Duplicate Elimination | Deduplication | Share common primitives |
| Cycle Detection | Circular dependency error | Modeling error detection |
| Type Compatibility | Version constraint checking | Engineering type system |
| Leaf Identification | Packages with no deps | Evidence-source primitives |
| Graph Assembly | Lockfile generation | Complete PDG output |

## 7.9 Primitive Dependency Graph Is Executable Grammar

Programming languages define syntax. Compilers parse source code into abstract syntax trees (ASTs). An AST represents the grammatical structure of a program — the hierarchical organization of expressions, statements, and declarations that gives the program its meaning.

Dependency graphs play the analogous role for engineering reasoning. Just as an AST represents the grammatical structure of source code, **Primitive Dependency Graphs represent the grammatical structure of engineering reasoning**. The nodes are the "vocabulary" (primitives), the edges are the "grammar rules" (dependencies), and the graph as a whole is the "sentence" (reasoning process).

But there is a crucial difference. A programming language AST is a passive data structure — it describes but does not execute. A Primitive Dependency Graph, by contrast, is **executable grammar**. It is not merely a description of reasoning structure; it is a specification that can be compiled into executable code. The graph simultaneously serves three roles:

1. **Documentation**: It explains the engineering reasoning process in a form readable by both humans and machines.
2. **Specification**: It defines precisely what computations are required and in what dependency order.
3. **Compilation Target**: It serves as the intermediate representation that the Domain Compiler transforms into optimized execution code.

This tripartite role — document, spec, and compilation target — is what makes the Primitive Dependency Graph the central organizing structure of TAE. Everything before the graph (Responsibility, IR, Planner, Primitive) contributes to its construction. Everything after the graph (Compiler, Validation, Delivery) operates on its output.

> **The Central Role of the PDG**: If TAE were a compiler, the Primitive Dependency Graph would be its IR — the canonical intermediate representation that all earlier phases produce and all later phases consume. It is the point of maximum structural clarity in the entire architecture.

## 7.10 From Dependency Graph to Reasoning Compiler

At this point in the TAE architecture, engineering reasoning has become completely declarative. Let us trace the transformation:

- **Responsibilities** have been decomposed into answerable engineering questions.
- **Questions** have been refined into testable hypotheses.
- **Hypotheses** have been grounded in specific evidence requirements.
- **Evidence requirements** have been mapped to engineering primitives.
- **Primitives** have been organized into typed dependency graphs.

The chain of abstraction is complete: from vague engineering intent to precise, machine-readable reasoning structure. However, the graph itself remains an **engineering specification**. It describes *what* must be computed and *why* dependencies exist, but it has not yet become an **executable program**. It declares reasoning structure but does not itself reason.

The next chapter introduces the **Domain Reasoning Compiler** — the component whose responsibility is to transform Primitive Dependency Graphs into optimized execution graphs suitable for runtime scheduling. Where the Dependency Graph answers "what depends on what?", the Compiler answers "how should it be executed?" This transformation — from declarative specification to imperative execution — completes the bridge between engineering thinking and engineering doing.

| TAE Component | Question Answered | Output Form |
|---|---|---|
| Responsibility Model | What is the engineering goal? | Structured objectives |
| Engineering IR | What does the solution require? | Declarative specification |
| Reasoning Planner | How should we reason about it? | Hypothesis-evidence pairs |
| Engineering Primitive | What is the smallest unit of reasoning? | Typed, documented primitives |
| **Dependency Graph** | **What depends on what?** | **Typed, acyclic graph** |
| **Domain Compiler (next)** | **How should it execute?** | **Optimized execution plan** |
