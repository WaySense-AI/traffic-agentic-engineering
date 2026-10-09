---
layout: default
title: "18 · Traffic Agent Runtime — Executing Engineering Reasoning"
nav_order: 28
---

# Chapter 18: Traffic Agent Runtime — Executing Engineering Reasoning

## 18.1 From Compilation to Execution

The Domain Reasoning Compiler (Chapters 11 and 16) transforms engineering responsibilities into Primitive Dependency Graphs. The Domain Primitive Library (Chapter 17) provides the building blocks. But a compiled PDG is just a **plan**—a static specification of what should be computed, in what order, with what dependencies.

The **Traffic Agent Runtime (TAR)** is the engine that **executes** that plan. It takes the PDG produced by the DRC and runs it: invoking primitives in the correct order, passing data between them, handling errors, managing state, collecting evidence, and producing deliverables.

If the DRC is the compiler, the TAR is the **virtual machine**—the runtime environment that gives life to compiled engineering reasoning.

> **The TAR does not compute anything itself. It orchestrates computation. Every engineering calculation happens inside a primitive (Chapter 17). The TAR's job is to ensure those primitives execute correctly, in the right sequence, with the right data, and produce auditable results.**

This chapter details the architecture, mechanisms, and operational characteristics of the Traffic Agent Runtime.

---

## 18.2 Runtime Architecture Overview

### 18.2.1 Layered Architecture

The TAR implements a layered architecture that separates concerns at each level:

```
┌─────────────────────────────────────────────┐
│              Application Layer               │
│    (User Interface / API Gateway)            │
├─────────────────────────────────────────────┤
│             Orchestration Layer              │
│   (Execution Engine / Scheduler / Monitor)   │
├───────────────────┬─────────────────────────┤
│   Primitive        │     Validation          │
│   Execution        │     Engine              │
│   Container        │                         │
├───────────────────┴─────────────────────────┤
│             State Management                 │
│   (Working Memory / Evidence Store / Cache)  │
├─────────────────────────────────────────────┤
│           Integration Layer                  │
│  (Data Sources / Simulators / External APIs) │
└─────────────────────────────────────────────┘
```

Each layer has distinct responsibilities:

| Layer | Responsibility | Key Components |
|---|---|---|
| **Application** | Accept requests, return results | REST API, WebSocket, CLI |
| **Orchestration** | Execute PDG, manage workflow | Scheduler, Executor, Monitor |
| **Primitive Execution** | Run individual primitives | Container, Sandbox, Resource Manager |
| **Validation** | Enforce validation rules | Pre-checker, Post-checker, Cross-validator |
| **State Management** | Maintain execution state | Working memory, Evidence store, Cache |
| **Integration** | Connect to external systems | Data adapters, Simulator wrappers, API clients |

### 18.2.2 Core Components

The TAR consists of six core components working in concert:

| Component | Role | Interface |
|---|---|---|
| **Graph Executor** | Traverses PDG and dispatches primitive invocations | `execute(PDG) -> ExecutionResult` |
| **Primitive Invoker** | Calls individual primitive implementations | `invoke(PrimitiveSpec, Inputs) -> Output` |
| **Data Manager** | Routes data between primitives, manages typing | `get(node_id) -> TypedValue`, `put(node_id, value)` |
| **Validation Engine** | Runs pre/post validation checks on primitive results | `validate(primitive, result) -> ValidationResult` |
| **Evidence Collector** | Captures provenance metadata throughout execution | `record(event) -> EvidenceEntry` |
| **Error Handler** | Detects, classifies, and recovers from errors | `handle(error) -> RecoveryAction` |

These components interact through well-defined interfaces, enabling independent testing, replacement, and evolution of each component.

---

## 18.3 The Execution Model

### 18.3.1 Dependency-Driven Traversal

The TAR executes a PDG by dependency rather than by sequence: a node becomes runnable the moment everything it depends on has produced its value. There is no master schedule to follow, and no fixed order baked into the graph.

Execution therefore proceeds as a loop with a simple shape:

1. **Seed the ready set.** Every node with no outstanding dependencies is runnable to begin with.
2. **Dispatch.** Each runnable node gathers its inputs from the working state, invokes its primitive, and produces either a value or a failure.
3. **Record.** A successful result is stored against the node and written to the evidence log; its dependents are re-examined and may become runnable in turn.
4. **Handle failure where it happens.** A failed node is retried, skipped, or escalated according to its class — a node that a documented default can satisfy does not stop the graph, while a node the deliverable genuinely rests on does.
5. **Assemble.** When nothing remains runnable, the deliverable is assembled from the completed nodes and the evidence log is closed.

The characteristics that matter to an engineer are consequences of this shape rather than features bolted onto it:

- **Data-driven.** Nodes run when their inputs are ready, not when a scheduler reaches them.
- **Parallel-capable.** Every node in the ready set may run concurrently.
- **Fault-tolerant.** A failure is contained to the node that failed; whether the graph continues or stops is a deliberate outcome, not an accident.
- **Traceable.** Every operation lands in the evidence log, in the order it happened.


### 18.3.2 Execution Strategies

The TAR supports multiple execution strategies depending on the nature of the PDG:

| Strategy | When Used | Characteristics |
|---|---|---|
| **Sequential** | Small graphs (< 10 nodes), strict ordering required | Simple, predictable, easy to debug |
| **Parallel** | Large graphs with many independent subgraphs | Faster utilization, requires thread-safe state management |
| **Streaming** | Continuous/real-time tasks with ongoing data arrival | Processes data as it arrives, unbounded execution time |
| **Incremental** | Re-execution after partial input changes | Only re-runs affected subgraph, caches unchanged results |

The strategy selection can be **automatic** (based on graph size and structure) or **explicit** (specified by the engineer or system configuration).

### 18.3.3 Scheduling and Prioritization

When multiple nodes are ready simultaneously, the scheduler decides execution order based on:

| Priority Factor | Rationale | Example |
|---|---|---|
| **Critical path length** | Nodes on longest path should start first | Saturation flow before delay |
| **Downstream fan-out** | High-fan-out nodes feed more dependents | Volume extraction before everything else |
| **Resource requirements** | Heavy computations scheduled during low-load periods | Simulation runs overnight |
| **User priority** | Engineer-marked critical steps | Manual override for key analyses |
| **Historical duration** | Past execution times predict future needs | Known-slow primitives get earlier slots |

The scheduler uses a **weighted combination** of these factors to produce a prioritized execution order that minimizes total completion time while respecting all dependencies.

---

## 18.4 Primitive Invocation Mechanism

### 18.4.1 The Invocation Lifecycle

Each primitive invocation follows a strict lifecycle:

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  RESOLVE │ → │ PREPARE  │ → │ EXECUTE  │ → │ VALIDATE │ → │  STORE   │
│          │   │          │   │          │   │          │   │          │
│ Find impl│   │ Gather   │   │ Run algo │   │ Check    │   │ Cache    │
│ Load code│   │ inputs   │   │ Compute  │   │ results  │   │ output   │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
     │              │             │             │             │
     ▼              ▼             ▼             ▼             ▼
 Implementation   Type-check    Sandboxed    Error/Warning  Provenance
 lookup           Coercion      execution    generation    attachment
```

### 18.4.2 Resolution: Finding the Right Implementation

A single primitive spec may have **multiple implementations**:

```yaml
primitive: CalculateSaturationFlowRate
implementations:
  - id: hcm_default
    type: builtin
    language: python
    source: "tae_primitives/hcm/saturation_flow.py"
    priority: 100

  - id: field_measurement
    type: external
    language: python
    source: "agency_modules/field_sfr.py"
    trigger: "sufficient_detector_data"
    priority: 90

  - id: local_regression
    type: external
    language: R
    source: "agency_models/sfr_regression.R"
    trigger: "local_calibration_available"
    priority: 80
```

The resolver selects the best implementation based on:
1. **Trigger conditions**: Is the required data/method available?
2. **Priority ranking**: When multiple qualify, pick highest priority
3. **User preference**: Explicit selection overrides automatic resolution
4. **Past performance**: Historical success rate influences future selections

### 18.4.3 Preparation: Input Gathering and Type Checking

Before execution, the TAR performs:

```
PREPARATION CHECKLIST:
□ All required inputs present? → Error if missing
□ Optional inputs defaulted? → Apply specified defaults
□ Types match specification? → Attempt coercion if compatible
□ Assumptions satisfied? → Flag warnings if violated
□ Constraints propagated? → Pass downstream constraints to input validator
□ Resources available? → Check memory, licenses, external service availability
```

Any failure here produces a **pre-execution error** that prevents the primitive from running with bad inputs—a fundamentally different approach from "garbage in, garbage out."

### 18.4.4 Execution: Sandboxed Computation

Primitives execute within a **sandboxed environment** that provides:

| Sandbox Feature | Purpose | Implementation |
|---|---|---|
| **Resource limits** | Prevent runaway computations | CPU time limit, memory cap |
| **Network isolation** | Control external access | Whitelisted endpoints only |
| **Filesystem isolation** | Protect host system | Temporary directory only |
| **Timeout enforcement** | Guarantee responsiveness | Per-primitive timeout (configurable) |
| **Logging capture** | Ensure complete audit trail | stdout/stderr captured to evidence store |

Sandboxing ensures that a buggy or malicious primitive cannot compromise the TAR or the host system—a critical property when primitives are contributed by third parties (Section 17.9.3).

### 18.4.5 Post-Execution: Validation and Storage

After execution completes:

1. **Validation Engine** runs all post-checks defined in the primitive spec (range checks, sanity checks, consistency checks)
2. **Results are typed** according to the output specification (`Derived(T)`, `Estimated(T)`, etc.)
3. **Provenance metadata** is attached (timestamp, implementation used, inputs hash, duration, resource usage)
4. **Results are stored** in the working memory, indexed by node ID for downstream consumption
5. **Dependents are notified** that their inputs are ready

---

## 18.5 State Management

### 18.5.1 The Working Memory Model

The TAR maintains a **working memory** that holds all intermediate and final results during execution:

```
WorkingMemory {
  nodes: Map<NodeID, TypedValue>,
  metadata: Map<NodeID, ProvenanceRecord>,
  status: Map<NodeID, NodeStatus>,  // pending/running/completed/failed/skipped
  dependencies: DependencyGraph,
  execution_log: EventLog[]
}
```

Key design decisions:

| Decision | Rationale |
|---|---|
| **Immutable results** | Once a node completes, its output cannot be modified (prevents subtle bugs) |
| **Typed storage** | Values carry their types through execution; type mismatches caught at consumption |
| **Lazy evaluation** | Results computed only when first requested by a dependent (for expensive operations) |
| **Eager caching** | Frequently-used values cached after first computation |
| **Automatic cleanup** | Intermediate values released when no longer needed (memory management) |

### 18.5.2 Evidence Collection

Every event during execution is recorded in an **evidence store**:

| Event Type | Data Recorded | Retention Policy |
|---|---|---|
| `PRIMITIVE_INVOKED` | Node ID, inputs hash, timestamp, implementation ID | Full retention |
| `PRIMITIVE_COMPLETED` | Outputs, duration, resource usage, validation results | Full retention |
| `PRIMITIVE_FAILED` | Error type, message, partial outputs (if any), stack trace | Full retention |
| `VALIDATION_CHECK` | Check name, condition, result (pass/fail/warning/error) | Full retention |
| `DATA_ACCESSED` | Source, query, records returned, timestamp | Summary (full on demand) |
| `DECISION_MADE` | Decision point, options considered, choice made, rationale | Full retention |
| `STATE_SNAPSHOT` | Complete working memory at checkpoint | Configurable (default: milestones only) |

This evidence store is the foundation for:
- **Explainability**: "Why did we get this answer?" → Trace through evidence
- **Auditability**: "Was this analysis done correctly?" → Review evidence chain
- **Reproducibility**: "Can we get the same result again?" → Replay from evidence log
- **Debugging**: "Where did things go wrong?" → Examine failure events

### 18.5.3 Checkpointing and Recovery

For long-running executions (large network studies, simulation-heavy analyses), the TAR supports **checkpointing**:

```
Checkpoint Trigger Conditions:
✓ After every N primitive completions (default: N=10)
✓ Before expensive operations (simulation runs, external API calls)
✓ On explicit user request
✓ At predefined milestone nodes (e.g., after LOS evaluation)

Checkpoint Contents:
✓ Complete working memory snapshot
✓ Execution position in PDG (which nodes done, which pending)
✓ Evidence log up to checkpoint time
✓ System state (resource usage, timestamps)

Recovery Process:
1. Load most recent checkpoint
2. Verify integrity (hash comparison)
3. Resume execution from checkpoint position
4. Append new events to existing evidence log
```

Checkpointing enables **resilience** against system failures, allowing long-running tasks to survive crashes, updates, and maintenance windows without losing work.

---

## 18.6 Error Handling and Fault Tolerance

### 18.6.1 Error Taxonomy

The TAR distinguishes several categories of errors, each with different handling strategies:

| Error Category | Example | Severity | Default Handling |
|---|---|---|---|
| **Input Missing** | Required volume data not available | Error | Request from user; block dependent nodes |
| **Type Mismatch** | String passed where number expected | Error | Attempt coercion; fail if impossible |
| **Constraint Violation** | Calculated cycle time exceeds maximum | Warning + Flag | Record violation; continue with warning |
| **Resource Exhaustion** | Out of memory during simulation | Error | Free resources; retry with reduced scope |
| **External Failure** | Simulator service unavailable | Retryable | Exponential backoff; fallback method |
| **Timeout** | Primitive exceeded time limit | Error | Terminate; report partial results |
| **Validation Failure** | Post-check detected invalid output | Error | Do not propagate; mark node failed |
| **Assertion Failure** | Internal invariant violated | Fatal | Abort entire execution |

### 18.6.2 Recovery Strategies

When an error occurs, the error handler selects a recovery strategy:

| Strategy | When Applicable | Effect |
|---|---|---|
| **Retry** | Transient failures (network timeout, temporary overload) | Re-invoke same primitive with same inputs |
| **Fallback** | Primary implementation unavailable | Switch to alternative implementation |
| **Substitution** | Input data missing but estimable | Use estimated/default value with warning flag |
| **Degradation** | Cannot achieve full precision | Continue with reduced accuracy; document limitation |
| **Skip** | Non-critical primitive fails | Mark skipped; use default/empty for dependents |
| **Abort** | Critical primitive fails; no recovery possible | Stop execution; produce partial deliverable with error report |
| **Escalate** | Requires human judgment | Pause execution; notify engineer for decision |

The recovery strategy can be **pre-configured** in the primitive spec (e.g., "if external simulator fails, fall back to HCM approximation") or **determined dynamically** by the error handler based on error type and context.

### 18.6.3 Graceful Degradation

A key design principle: **partial results are better than no results**. If a non-critical primitive fails, the TAR continues execution rather than aborting:

```
Scenario: Queue estimation primitive fails (detector malfunction)

Without graceful degradation:
  Entire analysis ABORTS → No deliverable produced → Engineer gets nothing

With graceful degradation:
  Queue estimation SKIPPED → Use historical queue data (flagged as estimate)
  ↓
  Delay calculation continues (with uncertainty increased)
  ↓
  LOS evaluation proceeds (with caveat about queue data quality)
  ↓
  Deliverable produced WITH WARNINGS about degraded components
```

The final deliverable explicitly marks which portions used **degraded or estimated data**, allowing engineers to assess whether the results remain fit for purpose despite imperfections.

---

## 18.7 Parallel and Distributed Execution

### 18.7.1 Intra-Graph Parallelism

Within a single PDG, the TAR automatically identifies and exploits parallelism:

```
Original Sequential Execution:
  [ExtractVolumes] → [CalculateSFR] → [CalculateCapacity] → [CalculateXc]
  [ExtractGeometry] ────────────────────────────────────────────────┘
  [ExtractTiming] ─────────────────────────────────────────────────→ [AllocateGreen]
  [GetConstraints] ────────────────────────────────────────────────→ [EvaluateLOS]

Parallel Execution (same PDG):
  Thread 1: [ExtractVolumes] ──→ [CalculateSFR] ─┐
  Thread 2: [ExtractGeometry] ────────────────────┤──→ [CalculateCapacity]
  Thread 3: [ExtractTiming] ─────────────────────┤         │
  Thread 4: [GetConstraints] ────────────────────┘         │
                                                          ↓
                                              [CalculateXc] → [AllocateGreen] → [EvaluateLOS]
```

Speedup depends on the graph structure:
- **Highly parallel graphs** (many independent branches): near-linear speedup with core count
- **Sequential chains** (long critical paths): limited speedup regardless of core count
- **Typical traffic engineering tasks**: 2-4x speedup on 8-core hardware

### 18.7.2 Inter-Graph Parallelism: Multi-Responsibility Execution

The TAR can execute **multiple independent responsibilities simultaneously**:

| Scenario | Example | Parallelism Benefit |
|---|---|---|
| **Multi-intersection study** | Optimize timing at 10 intersections | Each intersection analyzed independently |
| **Multi-strategy analysis** | Diagnose AND evaluate same corridor | Different strategies share observation layer |
| **Scenario comparison** | Evaluate 5 alternative designs | Same base data, different parameters |
| **Batch processing** | Generate weekly reports for 50 signals | Identical template, different data |

The TAR coordinates shared resources (data connections, simulation licenses, cache) across concurrent executions while maintaining isolation between them.

### 18.7.3 Distributed Execution

For very large-scale analyses (regional network optimization with hundreds of intersections), the TAR supports **distributed execution** across multiple machines:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Worker 1   │  │  Worker 2   │  │  Worker 3   │
│ Intersection│  │ Intersection│  │ Intersection│
│ A1-A20      │  │ A21-A40     │  │ A41-A60     │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │
       └────────────────┼────────────────┘
                        │
               ┌────────▼────────┐
               │  Coordinator    │
               │  (Aggregates    │
               │   results,      │
               │   manages deps) │
               └─────────────────┘
```

Distribution is transparent to the engineer—the TAR handles partitioning, communication, fault tolerance, and result aggregation automatically.

---

## 18.8 Interaction Patterns

### 18.8.1 Synchronous Request-Response

The simplest interaction pattern: send a responsibility, receive a deliverable.

```
Engineer → TAR: "Optimize timing at intersection A"
TAR: [compiles → executes → validates → assembles]
TAR → Engineer: TimingPlanDocument (with full evidence trail)
```

Latency: seconds to minutes (depending on complexity). Suitable for interactive use.

### 18.8.2 Asynchronous Long-Running Tasks

For complex analyses that take longer:

```
Engineer → TAR: "Evaluate corridor-wide signal coordination (15 intersections)"
TAR → Engineer: TaskID: task-abc123, Status: QUEUED
...
TAR → Engineer (callback): Status: PROGRESS, Complete: 8/15, ETA: 12 min
...
TAR → Engineer (callback): Status: COMPLETE
Engineer → TAR: GET /tasks/task-abc123/result
TAR → Engineer: CorridorEvaluationReport
```

Supports progress tracking, cancellation, and result retrieval at convenience.

### 18.8.3 Streaming Mode

For continuous monitoring and real-time response:

```
Engineer → TAR: "Monitor intersection A for oversaturation conditions"
TAR: [establishes continuous observation loop]

[Stream of situation assessments...]
TAE → Engineer: Situation=Normal (14:00)
TAE → Engineer: Situation=Congested (14:23) ⚠️
TAE → Engineer: Situation=Oversaturated (14:47) 🔴
TAE → Engineer: Recommendation: Increase cycle by 10s (auto-generated)

Engineer → TAR: ACK / MODIFY / OVERRIDE
```

Streaming mode connects the TAR directly to real-time data sources and produces continuous engineering intelligence rather than one-off analyses.

### 18.8.4 Interactive Collaboration Mode

For scenarios requiring human-in-the-loop judgment:

```
TAR → Engineer: "Conflicting evidence: detector data suggests v/c=0.95,
                but observed queues indicate v/c > 1.1.
                Which source do you trust more?"

Engineer → TAR: "Trust observed queues; detectors may be miscalibrated"

TAR: [adjusts reasoning, re-runs affected branch]
TAR → Engineer: Revised analysis using queue-based v/c=1.15,
                recommends immediate capacity improvement
```

Interactive mode pauses execution at decision points (marked by `dec:` primitives, Section 17.3.1) and resumes upon human input.

---

## 18.9 Performance Characteristics

### 18.9.1 Latency Benchmarks

| Task Type | Typical Node Count | Median Latency | P99 Latency |
|---|---|---|---|
| Single intersection diagnosis | 12-18 | 3-8 seconds | 15 seconds |
| Single intersection optimization | 22-35 | 10-30 seconds | 60 seconds |
| Corridor analysis (10 intersections) | 80-150 | 2-5 minutes | 10 minutes |
| Network-wide study (50+ intersections) | 300-800 | 15-45 minutes | 90 minutes |
| Real-time streaming update | 5-10 | < 500 ms | < 2 seconds |

Latency scales **sub-linearly** with problem size due to parallelization and caching of shared computations.

### 18.9.2 Resource Utilization

| Resource | Typical Consumption | Scaling Behavior |
|---|---|---|
| **Memory** | 50-500 MB per execution | Linear with graph size |
| **CPU** | 1-8 cores utilized | Parallelism-limited by graph structure |
| **I/O** | Database queries, file reads | Dominated by data access patterns |
| **External services** | Simulator calls, API requests | Varies by task type |
| **Storage** | Evidence logs (10-100 MB per task) | Linear with execution duration |

### 18.9.3 Optimization Techniques

The TAR employs several optimizations to maintain responsive performance:

| Technique | Description | Impact |
|---|---|---|
| **Result caching** | Cache primitive outputs keyed by inputs hash | 30-50% cache hit rate for repeated analyses |
| **Incremental execution** | On input change, only re-execute affected subgraph | 70-90% faster than full re-execution |
| **Lazy evaluation** | Defer computation until output actually needed | Reduces unnecessary work |
| **Pre-fetching** | Anticipate likely-needed data during idle periods | Masks I/O latency |
| **Parallelism** | Execute independent nodes concurrently | 2-4x speedup typical |
| **Approximation tiers** | Offer fast-approximate vs slow-precise options | User-controlled latency/accuracy tradeoff |

---

## 18.10 Integration with External Systems

### 18.10.1 Data Source Adapters

The TAR connects to diverse data sources through standardized adapters:

| Data Source | Adapter Protocol | Data Format | Refresh Rate |
|---|---|---|---|
| Traffic detector systems | MQTT / REST API | Time-series counts | Real-time (1-5 min) |
| Video analytics | RTSP / File import | Events, classifications | Near-real-time |
| GIS / CAD systems | WFS / File import | Geometry, topology | On-demand |
| Historical databases | SQL / Parquet | Archived observations | Batch |
| Weather services | REST API | Conditions, forecasts | Hourly |
| Incident management | REST / Webhook | Events, durations | Event-driven |

Adapters normalize heterogeneous data into the **typed observation format** (`Observation(T)`) expected by primitives (Chapter 17).

### 18.10.2 Simulator Wrappers

External traffic simulators are wrapped as primitive implementations:

| Simulator | Wrapper Type | Communication | Typical Use |
|---|---|---|---|
| Synchro Studio | COM API / File I/O | Local process | Signal timing optimization |
| VISSIM | COM API / DLL | Local process | Microscopic analysis |
| Paramics | API / Modeller | Local/remote | Network-wide simulation |
| AIMSUN | API / Python | Local/remote | Mesoscopic/microscopic |
| Custom agency models | Direct integration | Library call | Proprietary methods |

The wrapper interface abstracts simulator-specific details, presenting a uniform `sim:Run*` primitive interface to the TAR.

### 18.10.3 Delivery Channels

Completed deliverables are routed through appropriate channels:

| Channel | Deliverable Type | Trigger |
|---|---|---|
| **File system** | Reports, documents, data exports | Automatic on completion |
| **Email** | Summary reports, alerts | Configurable subscription |
| **API callback** | Programmatic consumers | Webhook registration |
| **Dashboard** | Real-time metrics, KPIs | Streaming mode |
| **CMS / Document management** | Official records | Agency policy requirement |

Delivery routing is configured per-deliverable-type, ensuring results reach the right stakeholders through the right channels.

---

## 18.11 Monitoring and Observability

### 18.11.1 Runtime Metrics

The TAR exposes comprehensive runtime metrics:

| Metric Category | Examples | Granularity |
|---|---|---|
| **Throughput** | Tasks/minute, primitives/second | Per-task, rolling average |
| **Latency** | P50/P95/P99 execution time | Per-task-type histogram |
| **Errors** | Error rate by category, retry count | Rolling window |
| **Resource** | CPU%, memory, open connections | Real-time gauge |
| **Queue depth** | Pending tasks, waiting primitives | Real-time counter |
| **Cache performance** | Hit rate, eviction rate, size | Rolling average |

Metrics are exposed via **Prometheus** format for integration with standard monitoring stacks (Grafana, Alertmanager).

### 18.11.2 Distributed Tracing

Each execution receives a unique **trace ID** that propagates through all primitive invocations, enabling end-to-end request tracing:

```
Trace: exec-a1b2c3d4
├── Span: DRC.compile (duration: 85ms)
├── Span: TAR.execute (duration: 23.4s)
│   ├── Span: obs:ExtractVolumes (duration: 1.2s)
│   │   └── Span: DB.query_detector_data (duration: 900ms)
│   ├── Span: drv:CalculateSaturationFlow (duration: 45ms)
│   ├── Span: drv:CalculateCapacity (duration: 32ms)
│   ├── Span: drv:AllocateGreenTimes (duration: 180ms)
│   │   └── Span: webster_optimize (duration: 150ms)
│   ├── Span: val:ValidateTimingPlan (duration: 88ms)
│   └── Span: del:GenerateTimingReport (duration: 340ms)
│       └── Span: template_render (duration: 280ms)
└── Span: delivery.route (duration: 120ms)
```

Tracing integrates with **OpenTelemetry** standards, enabling correlation with infrastructure-level observability.

### 18.11.3 Health Checks

The TAR exposes health endpoints for operational monitoring:

| Endpoint | Purpose | Response |
|---|---|---|
| `/health/live` | Liveness probe | 200 if process running |
| `/health/ready` | Readiness probe | 200 if accepting tasks |
| `/health/components` | Component-level status | Per-component healthy/degraded/down |
| `/health/version` | Version info | Build, registry version, last updated |

These support standard **Kubernetes** liveness/readiness probe patterns for containerized deployments.

---

## 18.12 Summary: The Runtime That Makes TAE Alive

The Traffic Agent Runtime is the operational heart of TAE. Without it, the DRC's compiled PDGs would remain inert specifications; the primitive library's knowledge would stay dormant; the carefully engineered validation rules would never fire.

Key principles from this chapter:

1. **Layered architecture** separates orchestration, execution, validation, state, and integration concerns.
2. **Dependency-driven traversal** executes PDGs correctly and efficiently, handling arbitrary graph structures.
3. **Primitive invocation lifecycle** (resolve → prepare → execute → validate → store) ensures correctness at every step.
4. **Sandboxed execution** isolates primitives for security and reliability.
5. **Comprehensive state management** includes typed working memory, evidence collection, and checkpointing.
6. **Graceful degradation** produces useful partial results instead of failing completely.
7. **Parallel and distributed execution** scale from single intersections to regional networks.
8. **Multiple interaction patterns** serve synchronous, asynchronous, streaming, and collaborative use cases.
9. **Rich observability** via metrics, tracing, and health checks enables operational excellence.
10. **Standard integrations** connect to data sources, simulators, and delivery channels.

The TAR transforms TAE from an interesting theoretical framework into a **practical, deployable, operable system**—one that can genuinely assist traffic engineers in their daily work, not merely demonstrate AI capabilities in controlled settings.

---

## Tables in This Chapter

| # | Table Name | Section |
|---|---|---|
| 1 | TAR Layered Architecture | 18.2.1 |
| 2 | Core Components | 18.2.2 |
| 3 | Execution Strategies | 18.3.2 |
| 4 | Scheduling Priority Factors | 18.3.3 |
| 5 | Invocation Lifecycle Stages | 18.4.1 |
| 6 | Sandbox Features | 18.4.4 |
| 7 | Error Taxonomy | 18.6.1 |
| 8 | Recovery Strategies | 18.6.2 |
| 9 | Parallel Execution Speedup | 18.7.1 |
| 10 | Interaction Patterns | 18.8 |
| 11 | Latency Benchmarks | 18.9.1 |
| 12 | Optimization Techniques | 18.9.3 |
| 13 | Data Source Adapters | 18.10.1 |
| 14 | Simulator Wrappers | 18.10.2 |
| 15 | Runtime Metrics Categories | 18.11.1 |
| 16 | Distributed Tracing Example | 18.11.2 |

---

*Chapter 18 End*
