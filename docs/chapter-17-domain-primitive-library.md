---
layout: default
title: "17 · Domain Primitive Library — The Building Blocks of Engineering Reasoning"
nav_order: 27
---

# Chapter 17: Domain Primitive Library — The Building Blocks of Engineering Reasoning

## 17.1 The Atomic Unit of Engineering Intelligence

If the Domain Reasoning Compiler (Chapter 16) is the engine that transforms responsibilities into reasoning plans, then **engineering primitives are the fuel** that engine consumes. A primitive is the smallest reusable unit of engineering capability within a given domain—one that cannot be meaningfully decomposed further without losing its engineering semantics.

This definition carries profound implications:

> **An engineering primitive is not a software function. It is a unit of engineering knowledge, encoded in a form that machines can execute but engineers can understand, verify, and dispute.**

Where a software function implements an algorithm, an engineering primitive **embodies a method**—complete with its theoretical foundations, applicability conditions, input requirements, output semantics, known limitations, and validation criteria. The difference is the difference between code and knowledge.

This chapter provides a comprehensive treatment of the Domain Primitive Library: what primitives are, how they are specified, organized, composed, and maintained as a living engineering asset.

---

## 17.2 Defining Engineering Primitives

### 17.2.1 Formal Definition

An engineering primitive `P` is defined as a tuple:

```
P = (name, category, inputs, outputs, assumptions, constraints,
     method, validation, provenance, strategy_applicability)
```

| Component | Description | Example (Saturation Flow) |
|---|---|---|
| `name` | Canonical identifier | `CalculateSaturationFlowRate` |
| `category` | Type of primitive (derive/estimate/simulate/validate/decide) | `derivation` |
| `inputs` | Typed input specifications | `Observation(Volume)`, `Reference(LaneConfig)` |
| `outputs` | Typed output specifications | `Derived(SaturationFlow)` |
| `assumptions` | Preconditions that must hold | "Stable flow conditions" |
| `constraints` | Boundary conditions on outputs | "1450 ≤ SFR ≤ 2500 veh/h/ln" |
| `method` | Algorithm or procedure used | HCM 6th Edition Method |
| `validation` | Correctness checks to apply | Range check, sanity check |
| `provenance` | Source authority | HCM Chapter 19 / Local calibration |
| `strategy_applicability` | Which strategies use this | diagnosis, design, evaluation |

Every component is **explicit**. Nothing is implicit, nothing is hidden, nothing relies on convention or programmer intuition. This explicitness is what makes primitives auditable, disputable, and improvable.

### 17.2.2 Primitives vs. Software Functions

The distinction between engineering primitives and software functions is fundamental to understanding TAE:

| Dimension | Software Function | Engineering Primitive |
|---|---|---|
| **Purpose** | Perform computation | Encode engineering method |
| **Interface** | Parameter list | Typed I/O + assumptions + constraints |
| **Semantics** | Implementation-defined | Domain-defined (HCM, MUTCD, etc.) |
| **Correctness** | Unit tests pass | Produces engineering-valid results |
| **Error handling** | Exceptions, error codes | Engineering judgment required |
| **Composition** | Function calls | Dependency graph with type checking |
| **Documentation** | Code comments | Full specification document |
| **Evolution** | Refactoring | Method revision / new research |
| **Accountability** | Developer | Domain expert + source authority |

A software function for calculating saturation flow rate might look like:

```python
def saturation_flow(volume, lanes):
    return volume / lanes * 1800  # ???
```

An engineering primitive for the same calculation includes everything needed to determine whether this computation is appropriate, correct, and sufficient for the engineering task at hand.

### 17.2.3 The Indivisibility Principle

A primitive is **indivisible** within its domain—not because it cannot be broken into smaller steps algorithmically, but because doing so would lose engineering meaning. Consider:

```
CalculateDelay(HCMMethod, Volume, Capacity, ProgressionFactor)
```

This could be decomposed into:
1. Calculate v/c ratio
2. Look up uniform delay factor
3. Calculate uniform delay component
4. Look up incremental delay factor
5. Calculate incremental delay component
6. Sum components

But these sub-steps are **not independent engineering concepts**—they are implementation details of the HCM delay procedure. No traffic engineer would request "calculate incremental delay factor" as a standalone task. The atomic unit of engineering work is **delay estimation**, not its internal arithmetic.

The indivisibility principle ensures that the primitive library operates at the **right level of abstraction**—the level at which engineers actually think, communicate, and make decisions.

---

## 17.3 Taxonomy of Engineering Primitives

### 17.3.1 Category Classification

Primitives fall into six fundamental categories based on their role in the reasoning chain:

#### Category 1: Observation Primitives (`obs:`)

Extract structured data from raw observations.

| Primitive | Input | Output | Notes |
|---|---|---|---|
| `obs:ExtractVolumeCounts` | Detector logs | `Observation(VolumeMatrix)` | Peak hour extraction, missing data handling |
| `obs:ExtractSpeedData` | Detector/video | `Observation(SpeedDistribution)` | Spot speed vs travel speed |
| `obs:ExtractOccupancy` | Detector logs | `Observation(Occupancy)` | Time-series occupancy patterns |
| `obs:ExtractQueueLength` | Video/detector | `Observation(QueueEstimate)` | Maximum/average/persistent queue |
| `obs:ExtractEventData` | Incident logs | `IncidentRecord>` | Duration, type, location, impact |

Observation primitives are the **boundary between the physical world and the reasoning system**. They transform raw sensor data into typed engineering observations, applying quality checks and uncertainty quantification in the process.

#### Category 2: Derivation Primitives (`drv:`)

Apply deterministic transformations to produce derived quantities.

| Primitive | Input | Output | Method |
|---|---|---|---|
| `drv:CalculateSaturationFlow` | Volumes, lane config | `Derived(SaturationFlow)` | HCM Ch.19 / Field measurement |
| `drv:CalculateCapacity` | Saturation flows, phases | `Derived(Capacity)` | Critical lane sum / shared lane |
| `drv:CalculateDegreeOfSaturation` | Volume, capacity | `Derived(Xc)` | v/c ratio |
| `drv:CalculateDelay` | Xc, C, g/C, progression | `Derived(Delay)` | HCM uniform + incremental |
| `drv:CalculateLOS` | Delay, control type | `Derived(LOS)` | HCM LOS thresholds |
| `drv:CalculateQueueStorage` | Geometry, arrival pattern | `Derived(StorageRatio)` | Available vs required |

Derivation primitives are the **workhorses of traffic engineering computation**. They implement established methods from sources like the HCM, with clear inputs, deterministic outputs, and well-documented assumptions.

#### Category 3: Estimation Primitives (`est:`)

Infer quantities that cannot be directly measured.

| Primitive | Input | Output | Uncertainty Source |
|---|---|---|---|
| `est:EstimateODMatrix` | Counts, partial OD | `Estimated(ODMatrix)` | Underdetermined system |
| `est:EstimateTurningProportions` | Movement counts | `Estimated(TurningProp)` | Sampling error, time variation |
| `est:EstimateDemand` | Observed counts, adjustment | `Estimated(Demand)` | Peaking factors, growth |
| `est:EstimateFutureVolume` | Historical, land use | `Estimated(ProjectedVolume)` | Land use model uncertainty |
| `est:EstimateSaturationFlowDefault` | Area type, facility type | `Estimated(SFR)` | Default values, local variation |

Estimation primitives explicitly **quantify uncertainty**. Their outputs are always `Estimated(T)` types, never `Derived(T)` types—the distinction reminds everyone that inference was involved.

#### Category 4: Simulation Primitives (`sim:`)

Execute traffic simulation models.

| Primitive | Input | Output | Model |
|---|---|---|---|
| `sim:RunSynchro` | Network, timing, volumes | `Simulated(Performance)` | Synchro Studio |
| `sim:RunVISSIM` | Network, behavior params | `Simulated(MicroMetrics)` | PTV VISSIM |
| `sim:RunParamics` | Network, demand matrix | `Simulated(NetworkMetrics)` | Paramics |
| `sim:RunMesoscopic` | Corridor, simplified | `Simulated(CorridorLevel)` | Fast mesoscopic model |

Simulation primitives wrap external simulation tools, providing a **uniform interface** regardless of which specific simulator is used. They also attach simulation metadata (model version, seed values, run duration) to all outputs.

#### Category 5: Validation Primitives (`val:`)

Verify correctness of other primitives' outputs.

| Primitive | Target | Check Type | Severity |
|---|---|---|---|
| `val:CheckDataQuality` | Observed data | Completeness, plausibility | Error/warning |
| `val:CheckPhysicalBounds` | Derived quantity | Physical feasibility | Error |
| `val:CheckConstraintSatisfaction` | Design result | Policy/engineering limits | Error/warning |
| `val:CrossValidate` | Multiple sources | Consistency across methods | Warning |
| `val:CheckHistoricalConsistency` | Result vs history | Reasonable change magnitude | Warning |
| `val:SanityCheckResult` | Final deliverable | Engineer-level reasonableness | Info/error |

Validation primitives are **never optional** in TAE—they are automatically inserted by the DRC based on strategy requirements (Chapter 16).

#### Category 6: Decision Primitives (`dec:`)

Apply engineering judgment where algorithms cannot decide.

| Primitive | Context | Output | Requires |
|---|---|---|---|
| `dec:SelectAnalysisMethod` | Data availability, precision needs | `Decision(MethodChoice)` | Trade-off judgment |
| `dec:SelectDesignCriteria` | Agency policy, context | `Decision(DesignStandard)` | Policy interpretation |
| `dec:ResolveConflictingEvidence` | Multiple sources disagree | `Decision(EvidenceWeighting)` | Expert judgment |
| `dec:DetermineScope` | Responsibility statement | `Decision(AnalysisBoundary)` | Scope judgment |
| `dec:ClassifySituation` | Observations, context | `Decision(SituationType)` | Pattern recognition |

Decision primitives represent the **irreducible core of human expertise**—points where engineering judgment cannot (yet) be fully automated. TAE makes these points **explicit** rather than hiding them inside opaque model weights.

### 17.3.2 Cross-Category Composition

Real engineering tasks compose primitives from multiple categories. Here's the typical composition pattern for a signal timing optimization task:

```
[obs:] ExtractVolumeCounts ─────┐
[obs:] ExtractGeometry ─────────┤
[obs:] ExtractSignalTiming ────┤
                                  ├─→ [drv:] CalculateSaturationFlow
[ref:] HCM_SaturationDefaults ──┤      │
                                  │      ├─→ [drv:] CalculateCapacity
[drv:] CalculateLaneGroups ──────┘      │
                                         │
[ref:] Webster_Method ──────────────────┤      │
                                         ├─→ [drv:] AllocateGreenTimes
[constr:] MinGreen_Policy ───────────────┤      │
[constr:] PedClearance_Requirements ────┘      │
                                                │
[val:] ValidateTimingPlan ◄────────────────────┘
        │
        ├─→ [drv:] EvaluateLOS
        │         │
│         └─→ [del:] GenerateTimingReport
│
└─→ [val:] CrossValidateWithSimulation
```

This composition involves **all six categories** working together—a pattern typical of non-trivial engineering tasks.

---

## 17.4 Primitive Specification Language

### 17.4.1 Complete Specification Template

Each primitive in the library has a complete machine-readable specification. Below is the full template with a concrete example:

```yaml
# ============================================================
# Primitive Specification: CalculateSaturationFlowRate
# ============================================================

meta:
  id: drv:CalculateSaturationFlowRate
  version: "2.1.0"
  last_updated: "2025-03-15"
  author: "TAE Core Team"
  reviewer: "Dr. [Traffic Engineering Expert]"
  status: active  # active | deprecated | experimental

classification:
  category: derivation
  subcategory: capacity_analysis
  hcm_reference: "HCM 6th Ed., Chapter 19, Exhibit 19-14"
  alternative_methods:
    - id: field_measurement
      reference: "Highway Capacity Manual, Appendix"
    - id: regression_model
      reference: "Local agency calibration study (2023)"

inputs:
  - name: approach_volumes
    display_name: "Approach Peak Hour Volumes"
    type: "Observation(Volume) | Estimated(Volume)"
    required: true
    description: "Peak hour approach volumes by movement (veh/h)"
    units: "veh/h"
    constraints:
      min: 0
      max: 5000
      sanity: "should be consistent with geometry capacity"

  - name: lane_configuration
    display_name: "Lane Configuration"
    type: "Reference(LaneConfiguration)"
    required: true
    description: "Number of lanes, lane use assignments, shared movements"
    units: N/A
    source: "field observation or as-built plans"

  - name: heavy_vehicle_percentage
    display_name: "Heavy Vehicle Percentage"
    type: "Observation(HV%) | Estimated(HV%)"
    required: false
    default: 0.02
    description: "Percentage of heavy vehicles in traffic stream"
    units: "%"
    range: [0, 100]

outputs:
  - name: saturation_flows
    display_name: "Saturation Flow Rates"
    type: "Derived(SaturationFlow)"
    description: "Saturation flow rate per lane group (veh/h/ln)"
    units: "veh/h/ln"
    shape: "array[lane_groups]"

assumptions:
  - id: stable_flow
    text: "Traffic flow is in stable (not breakdown) conditions"
    violation_handling: "flag warning; result may overestimate true SFR"
  - id: base_conditions
    text: "Base saturation flow assumed at 1900 pc/h/ln (passenger cars)"
    violation_handling: "apply adjustment factors if conditions differ"

constraints:
  - name: minimum_sfr
    expression: "output >= 1450"
    source: "HCM lower bound for signalized intersections"
    severity: error
    message: "Saturation flow below physically plausible minimum"

  - name: maximum_sfr
    expression: "output <= 2500"
    source: "Empirical upper bound under normal conditions"
    severity: warning
    message: "Unusually high saturation flow; verify data"

method:
  primary: hcm_default
  algorithm: |
    For each lane group LG:
      1. Determine base saturation flow: SFR_base = 1900 pc/h/ln
      2. Apply heavy vehicle adjustment: f_HV = 1/(1 + P_HV*(E_HV-1))
      3. Apply grade adjustment (if applicable): f_G = 1 - 0.005*grade%
      4. Apply other adjustments (parking, bus, area, lane width)
      5. SFR_LG = SFR_base × f_HV × f_G × f_P × f_B × f_A × f_W
    Return array of SFR per lane group

  alternatives:
    - name: field_measurement
      trigger: "sufficient detector data available"
      method: "Headway-based measurement from saturated green time"
    - name: local_calibration
      trigger: "local agency has calibrated model"
      method: "Use agency-specific regression equation"

validation:
  pre_checks:
    - name: data_completeness
      condition: "approach_volumes not null AND lane_configuration not null"
      severity: error
    - name: volume_sanity
      condition: "sum(approach_volumes) < 10000"
      severity: warning

  post_checks:
    - name: range_check
      condition: "ALL(1450 <= sfr <= 2500)"
      severity: error
    - name: consistency_check
      condition: "max(sfr)/min(sfr) < 3.0"
      severity: warning
    - name: comparison_to_defaults
      condition: "abs(sfr - hcm_default) < 30%"
      severity: info

evidence_requirements:
  - type: input_data
    retention: full_traceability
  - type: intermediate_values
    retention: key_adjustment_factors
  - type: assumptions_applied
    retention: list_of_assumptions_with_status

strategy_applicability:
  diagnosis: true       # Needed for capacity analysis
  design: true          # Core input to timing calculations
  evaluation: true      # Needed for LOS determination
  prediction: conditional # Only if predicting capacity
  control: false        # Not used in real-time control
  planning: true        # Used in long-term planning studies

dependencies:
  - target: approach_volumes
    type: data
  - target: lane_configuration
    type: reference
  - target: heavy_vehicle_percentage
    type: data_optional
  - target: hcm_adjustment_factors
    type: reference

known_limitations:
  - "Assumes uniform arrival patterns within analysis period"
  - "Less accurate for actuated signals with variable timing"
  - "Heavy vehicle passenger-car equivalent may vary locally"
  - "Does not account for spillback effects from downstream"

version_history:
  - version: "2.1.0"
    date: "2025-03-15"
    changes: "Added local_calibration alternative; updated HV PCE values"
  - version: "2.0.0"
    date: "2024-06-01"
    changes: "Restructured to new specification format; added evidence tracking"
  - version: "1.0.0"
    date: "2023-01-15"
    changes: "Initial specification"
```

This level of detail may seem excessive—but it is precisely this detail that enables the DRC to compile correct reasoning chains, the TAR to execute them reliably, and engineers to verify every step.

### 17.4.2 Specification Quality Criteria

Not all specifications are created equal. The library enforces quality gates:

| Criterion | Required | Checked By |
|---|---|---|
| Complete I/O typing | All inputs/outputs have types | Type checker |
| Explicit assumptions | At least one assumption listed | Linter |
| Constraint definitions | At least output range constraint | Validator |
| Validation rules | Both pre-check and post-check | Test harness |
| Source attribution | Primary method has authoritative reference | Reviewer |
| Strategy mapping | All 6 strategies addressed | Registry validator |
| Known limitations | At least one limitation documented | Reviewer |
| Version history | Initial + current version | Linter |

A primitive failing any **required** criterion cannot be registered in the active library. This rigor is what separates the TAE primitive library from a casual collection of utility functions.

---

## 17.5 The Primitive Registry Architecture

### 17.5.1 Registry Structure

The Domain Primitive Library is implemented as a **structured registry**—a database of primitive specifications with indexing, search, and dependency query capabilities:

```
Domain Primitive Registry
├── Core Primitives (stable, well-validated)
│   ├── Observation (25 primitives)
│   ├── Derivation (48 primitives)
│   ├── Estimation (22 primitives)
│   ├── Simulation (12 primitives)
│   ├── Validation (18 primitives)
│   └── Decision (15 primitives)
├── Extended Primitives (domain-specific)
│   ├── Signal Timing (35 primitives)
│   ├── Freeway Operations (28 primitives)
│   ├── Safety Analysis (19 primitives)
│   ├── Transit Operations (14 primitives)
│   ├── Active Transportation (11 primitives)
│   └── Emergency Management (8 primitives)
├── Experimental Primitives (under validation)
│   └── (varies by release)
└── Deprecated Primitives (archived, not executable)
    └── (historical record only)
```

Total: approximately **255 primitives** in the current registry, spanning all major areas of traffic engineering practice.

### 17.5.2 Indexing and Discovery

The registry supports multiple access patterns:

| Access Pattern | Mechanism | Use Case |
|---|---|---|
| **ByName** | Hash index on `id` | Direct lookup when primitive is known |
| **ByCategory** | B-tree index on `category` | Browse all derivation primitives |
| **ByStrategy** | Inverted index on `strategy_applicability` | Find primitives for design strategy |
| **ByInputType** | Inverted index on input types | Find primitives consuming `Observation(Volume)` |
| **ByOutputType** | Inverted index on output types | Find primitives producing `Derived(Delay)` |
| **ByKeyword** | Full-text search on name/description | Natural language discovery |
| **ByDependency** | Graph traversal | Find all ancestors/descendants of a primitive |
| **BySource** | Index on `method.primary` | Find all HCM-based primitives |

These indexes enable the DRC's dependency resolution (Chapter 16) to operate efficiently—even with hundreds of primitives, resolution completes in milliseconds.

### 17.5.3 Versioning and Lifecycle

Primitives follow a **semantic versioning** scheme with strict lifecycle stages:

```
Experimental (0.x.x)
    │  ▼
Proposed (review pending)
    │  ▼
Active (1.x.x, 2.x.x, ...)
    │  ▼
Deprecated (still usable, warnings issued)
    │  ▼
Retired (archived, no longer executable)
```

Transitions between stages require **formal review**:

| Transition | Trigger | Approver | Documentation Required |
|---|---|---|---|
| Experimental → Proposed | Internal testing complete | Tech lead | Test results, performance data |
| Proposed → Active | External review pass | Domain expert board | Peer review sign-off |
| Active → Deprecated | Superseded by better method | Registry maintainer | Migration guide, replacement ID |
| Deprecated → Retired | No dependents remain | System admin | Archive record, historical note |

This lifecycle management ensures that the registry remains **current, correct, and trustworthy**—essential properties for safety-critical engineering applications.

---

## 17.6 Primitive Composition Patterns

### 17.6.1 Linear Chains

The simplest composition pattern is a linear chain where each primitive's output feeds the next:

```
RawDetectorData → obs:ExtractVolumes → drv:CalculateSaturationFlow → drv:CalculateCapacity → drv:CalculateXc → drv:CalculateDelay → drv:EvaluateLOS
```

Linear chains are common for **straightforward analysis tasks** where one clearly-defined computation sequence produces the desired result. Most HCM operational analyses follow this pattern.

### 17.6.2 Fan-Out (One-to-Many)

A single primitive's output may feed multiple downstream primitives:

```
ApproachVolumes ─┬→ drv:CalculateSaturationFlow → drv:CalculateCapacity
                 ├→ est:EstimateTurningProportions → drv:CalculateLaneGroupVolumes
                 └→ val:CheckDataQuality → val:CheckTemporalConsistency
```

Fan-out occurs when the same input data serves multiple analytical purposes. The DRC optimizes fan-out by computing the shared input once and distributing it.

### 17.6.3 Fan-In (Many-to-One)

Multiple primitives' outputs converge into a single downstream primitive:

```
ObservedDelays ──┐
SimulatedDelays ─┼→ val:CrossValidate → dec:ResolveDiscrepancy
HistoricalDelays ─┘
```

Fan-in represents **multi-source evidence fusion**—a critical pattern for validation and decision-making.

### 17.6.4 Conditional Branches

Different execution paths based on runtime conditions:

```
IF data_quality == 'high':
    drv:CalculateSaturationFlow(observed_data)
ELSE IF data_quality == 'medium':
    est:EstimateSaturationFlow(observed_data + defaults)
ELSE:
    ref:GetDefaultSaturationFlow(area_type)
```

Conditional branches handle the **imperfect information** that characterizes real-world engineering. The DRC resolves branch conditions at compile time where possible; remaining runtime branches are handled by the TAR.

### 17.6.5 Iterative Loops

Some computations require iteration until convergence:

```
LOOP:
    drv:AllocateGreenTimes(current_timing)
    val:CheckCycleTimeConstraints(result)
    IF constraint_violated:
        adjust_parameters()
        CONTINUE
    ELSE:
        BREAK
```

Iterative loops are wrapped as **composite primitives**—the loop structure is internal; externally they present the same interface as any other primitive. This preserves the acyclic nature of the PDG while still supporting iterative algorithms.

---

## 17.7 The Dynamic Primitive Graph

### 17.7.1 Static Registry, Dynamic Graph

A crucial architectural principle: **the primitive registry is static, but the primitive graph is dynamic**. The registry contains all available primitives with their specifications. The graph is constructed on-demand for each specific engineering responsibility, selecting and connecting only the primitives that are relevant.

Consider two requests that sound similar but activate very different graphs:

| Request | Activated Primitives | Graph Size |
|---|---|---|
| "Why is intersection A congested?" | Diagnosis-focused: ~18 primitives | Medium |
| "Optimize timing at intersection A" | Design-focused: ~24 primitives | Large |
| "Will intersection A be OK next year?" | Prediction-focused: ~15 primitives | Medium |
| "Is the new timing plan working?" | Evaluation-focused: ~20 primitives | Large |

Same intersection, different questions, different graphs. This dynamism is what allows a single primitive library to serve the full spectrum of engineering tasks without combinatorial explosion.

### 17.7.2 Graph Construction Process

The graph construction process (executed by the DRC, Chapter 16) moves through five steps. Each step consumes what the previous one produced, and leaves the graph more determined than it found it.

1. **Classify the strategy.** The responsibility, the situation, and the context together establish which kind of engineering question is being asked. Strategy is settled first because it governs what counts as a complete answer.

2. **Identify seed primitives.** The responsibility statement is matched against the registry to produce the primitives that directly answer the question — for example `AllocateGreenTimes` and `EvaluateLOS` for a design request.

3. **Expand dependencies.** From each seed, the declared requirements are followed outward to the primitives that can satisfy them, and onward from there, until every requirement is either satisfied by supplied data or reported as missing. This expansion is what turns a short list of answers into a full reasoning chain.

4. **Insert validation.** Nodes for which the strategy demands verification receive a validation primitive downstream of them, so that verification is part of the graph rather than a step someone has to remember.

5. **Optimize.** Primitives whose output nothing consumes are removed, computations shared by several consumers are collapsed into one, and independent subgraphs are identified so they can run concurrently.

The resulting graph is **minimal** (no unnecessary primitives), **complete** (all dependencies satisfied), **valid** (type-checked and constraint-satisfied), and **executable** (can be directly run by the TAR).


### 17.7.3 Graph Properties

Every generated graph satisfies these properties:

| Property | Definition | Enforcement |
|---|---|---|
| **Acyclicity** | No circular dependencies (except controlled iterations) | Cycle detection during construction |
| **Completeness** | All transitive dependencies included | BFS expansion until frontier empty |
| **Type Safety** | All connections satisfy type compatibility | Type checker at each edge insertion |
| **Constraint Satisfaction** | All declared constraints checkable | Constraint propagation during construction |
| **Strategy Alignment** | Every primitive applicable to chosen strategy | Strategy filter at each inclusion |
| **Traceability** | Every node has provenance metadata | Automatic metadata attachment |

These properties are **invariants**—they hold for every graph produced by the DRC, regardless of the specific responsibility or domain.

---

## 17.8 Primitives and Explainability

### 17.8.1 Inherent Explainability

One of the most important properties of the primitive-based architecture is **inherent explainability**. Because every step in the reasoning chain corresponds to a named, documented, domain-meaningful primitive, the entire reasoning process can be explained in engineering terms:

**Opaque AI explanation:**
> "The model predicts a cycle time of 95 seconds based on learned patterns in the training data."

**TAE primitive-based explanation:**
> "The recommended cycle time of 95 seconds results from:
> 1. **Saturation flow calculation** (HCM Ch.19 method): 1850 veh/h/ln per critical lane group
> 2. **Critical lane sum**: 1,420 veh/h total critical volume
> 3. **Webster optimum formula**: Co = (1.5L + 5) / (1 - Y) = 92.3s
> 4. **Policy constraint**: Minimum cycle time = 90s (local agency standard)
> 5. **Rounding**: Nearest 5-second increment = 95s
>
> Key assumption: Heavy vehicle factor = 2.0 (default HCM value); actual HV% not measured."

The difference is stark. One explanation invites trust based on authority; the other invites **verification based on understanding**.

### 17.8.2 Explanation Generation

The TAR automatically generates explanations from the executed primitive graph:

| Explanation Level | Content | Audience |
|---|---|---|
| **Summary** | High-level conclusion + key drivers | Decision-makers |
| **Technical** | Step-by-step computation trace | Engineers |
| **Detailed** | Full PDG with all intermediate values | Auditors/reviewers |
| **Debug** | Execution log with timestamps and errors | Developers |

Each level is mechanically derivable from the same underlying execution—no separate explanation model needed. The primitive graph **is** the explanation.

---

## 17.9 Maintaining and Evolving the Primitive Library

### 17.9.1 Governance Model

The primitive library is a **shared engineering asset** that requires governance:

| Role | Responsibility | Authority |
|---|---|---|
| **Domain Experts** | Define new primitives, review proposed changes | Propose, approve content |
| **Registry Maintainers** | Manage versions, enforce quality gates | Merge, reject submissions |
| **System Administrators** | Deploy updates, manage infrastructure | Operational control |
| **End Users (Engineers)** | Report issues, suggest improvements | Feedback channel |
| **Agency Standards Bodies** | Set policy constraints, approve methods | Regulatory authority |

No single role has unchecked authority. Changes require consensus across relevant stakeholders.

### 17.9.2 Change Management Process

All modifications follow a structured workflow:

```
Proposal → Technical Review → Domain Review → Testing → Staging → Deployment → Monitoring
   │           │              │            │          │           │            │
   ▼           ▼              ▼            ▼          ▼           ▼            ▼
 Issue/Feature  Code/spec     Expert sign-off  Automated  Canary     Full roll-out  Performance
  description   quality gate   correctness     test suite   release     to production  metrics
```

Rollbacks are always possible—every deployment is tagged and can be reverted to the previous version within minutes.

### 17.9.3 Community Contribution

While the core library is maintained by the TAE team, **extended primitives** can be contributed by the user community:

1. Develop primitive following specification template
2. Submit to extended registry with test cases
3. Automated quality checks validate format compliance
4. Peer review assesses engineering correctness
5. Upon approval, primitive enters experimental stage
6. After successful field validation, promotion to active stage

This community model allows the library to **scale beyond what any single team could maintain**, while ensuring quality through rigorous review processes.

---

## 17.10 Summary: Primitives as the Foundation of Engineering AI

The Domain Primitive Library is the foundation upon which all other TAE components build. Without well-defined primitives, the DRC has nothing to compile; without the registry, the TAR has nothing to execute; without the graph structure, there is no explainability.

Key principles from this chapter:

1. **Primitives are engineering knowledge, not code.** Each primitive encodes a method with full semantic specification.
2. **Six categories** (observation, derivation, estimation, simulation, validation, decision) cover all engineering activities.
3. **Complete specifications** include I/O types, assumptions, constraints, methods, validation, and strategy applicability.
4. **The registry** organizes ~255 primitives with rich indexing for efficient lookup and composition.
5. **Five composition patterns** (linear, fan-out, fan-in, conditional, iterative) express all reasoning structures.
6. **Dynamic graph construction** selects relevant primitives per task—static registry, dynamic usage.
7. **Inherent explainability**—the primitive graph IS the explanation, no separate model needed.
8. **Governance and evolution** ensure the library remains current, correct, and community-enhanced.

The primitive library transforms traffic engineering from a craft passed down through apprenticeship into a **systematic, computable, and continuously improving body of engineering knowledge**. This is the foundation upon which engineering AI must be built.

---

## Tables in This Chapter

| # | Table Name | Section |
|---|---|---|
| 1 | Primitive Tuple Definition | 17.2.1 |
| 2 | Primitives vs Software Functions | 17.2.2 |
| 3 | Observation Primitives Catalog | 17.3.1 |
| 4 | Derivation Primitives Catalog | 17.3.1 |
| 5 | Estimation Primitives Catalog | 17.3.1 |
| 6 | Simulation Primitives Catalog | 17.3.1 |
| 7 | Validation Primitives Catalog | 17.3.1 |
| 8 | Decision Primitives Catalog | 17.3.1 |
| 9 | Specification Quality Criteria | 17.4.2 |
| 10 | Registry Structure Summary | 17.5.1 |
| 11 | Registry Access Patterns | 17.5.2 |
| 12 | Primitive Lifecycle Stages | 17.5.3 |
| 13 | Dynamic Graph Example | 17.7.1 |
| 14 | Graph Invariant Properties | 17.7.3 |
| 15 | Explanation Levels | 17.8.2 |
| 16 | Governance Roles | 17.9.1 |

---

*Chapter 17 End*
