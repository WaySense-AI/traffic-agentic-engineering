---
layout: default
title: "11 · Domain Reasoning Compiler"
nav_order: 21
---

# Chapter 11: Domain Reasoning Compiler

## From Engineering Knowledge to Executable Reasoning

Engineering knowledge becomes engineering intelligence only when it can be compiled into executable reasoning.

---

### 11.1 The Fundamental Problem: Engineering Knowledge Is Not Directly Executable

Traffic engineering has accumulated decades of valuable knowledge. This intellectual heritage spans multiple categories:

| Category | Examples | Form |
|----------|----------|------|
| **Engineering Specifications** | MUTCD, HCM, Signal Timing Guidelines | Regulatory documents |
| **Design Manuals** | AASHTO Green Book, ITE Traffic Engineering Handbook | Reference manuals |
| **Signal Timing Methods** | Webster's Method, HCM Platoon Dispersion, Actuated Control Algorithms | Procedural descriptions |
| **Traffic Flow Theory** | Fundamental Diagram, Shockwave Analysis, Queueing Theory | Mathematical frameworks |
| **Simulation Methodologies** | Microscopic, Mesoscopic, Macroscopic modeling approaches | Conceptual models |
| **Practical Experience** | Heuristics, rules of thumb, case-based reasoning | Tacit knowledge |

However, nearly all of this knowledge exists in a form that is fundamentally human-centric: **documents written for human engineers to read, interpret, and apply**.

Consider a concrete example. The Highway Capacity Manual (HCM) provides a detailed methodology for calculating the Level of Service (LOS) at a signalized intersection. An experienced traffic engineer can:

1. **Read** the HCM methodology description
2. **Understand** which parameters are required (saturation flow rates, geometric data, signal timing)
3. **Apply** the correct formulas in the proper sequence
4. **Interpret** the results within context
5. **Decide** what adjustments or follow-up analyses are needed

A machine cannot perform this sequence without explicit programming. The knowledge contained in the HCM is **descriptive** — it describes *what* an engineer should do — but it is not **executable** — it does not contain sufficient structure for a machine to automatically execute the methodology.

This observation reveals a critical gap in current AI-assisted engineering:

> **The knowledge exists. The computation exists. But the bridge between them — the mechanism that transforms descriptive engineering knowledge into executable reasoning processes — does not exist.**

Traffic Agentic Engineering introduces a new concept to address this gap: the **Domain Reasoning Compiler (DRC)**.

The purpose of DRC is not to compile programming languages into machine code. Its purpose is far more ambitious and domain-specific: **to compile engineering knowledge into executable reasoning processes**.

---

### 11.2 Why Prompt Engineering Is Not Enough

Before introducing the DRC architecture, it is essential to understand why existing approaches — particularly prompt engineering — fall short of achieving true engineering intelligence.

Many AI systems attempt to encode engineering knowledge inside natural language prompts. For example, a system prompt might instruct a language model as follows:

> "You are a traffic engineering assistant. When asked about intersection performance, evaluate queue length using the HCM methodology, calculate control delay using the Webster formula, and recommend signal timing adjustments based on capacity analysis results."

This approach has achieved notable success in conversational AI applications. However, when applied to genuine engineering responsibilities, it exhibits two fundamental limitations:

#### Limitation 1: Engineering Logic Becomes Implicit and Opaque

When engineering logic is embedded within natural language prompts, it becomes **implicit rather than explicit**. Consider the following scenario:

An engineer submits the responsibility: *"Diagnose why Intersection A experiences recurring afternoon peak congestion."*

A prompt-engineered system might produce a reasonable response — identifying potential causes such as insufficient capacity, spillback from downstream intersections, or suboptimal signal timing. However:

- **The reasoning path is opaque.** How did the system decide to examine capacity first, rather than signal timing? Was this decision principled or arbitrary?
- **The logic cannot be inspected.** There is no formal representation of the diagnostic reasoning that an engineer could review, modify, or validate.
- **The process cannot be reused systematically.** The same diagnostic logic embedded in the prompt cannot be extracted, versioned, tested independently, or composed with other reasoning processes.

In essence, prompt engineering treats engineering knowledge as **prose to be read by a language model**, rather than as **structure to be executed by an engineering system**.

#### Limitation 2: Prompts Cannot Express Explicit Engineering Dependencies

Engineering reasoning is inherently **dependency-driven**. The calculation of delay depends on queue length estimation, which depends on arrival pattern analysis, which depends on traffic count data. These dependencies form a directed graph that determines the correct execution order, identifies which intermediate results must be validated, and specifies which evidence sources are required at each step.

Natural language prompts cannot formally express these dependencies. A prompt may mention:

> "Analyze queue length, estimate delay, evaluate capacity, and recommend timing adjustments."

But it cannot formally define:
- That **queue analysis must precede** delay estimation
- That **delay estimation requires** saturation flow rate as input
- That **capacity evaluation constrains** the feasible range of timing adjustments
- That **timing recommendations must satisfy** pedestrian minimum green requirements

Without explicit dependency representation, engineering reasoning remains **implicit** — hidden within the statistical patterns of the language model rather than encoded as verifiable engineering logic.

#### The Consequence: Engineering Intelligence Without Engineering Rigor

These limitations lead to a critical consequence: **prompt-engineered systems can produce plausible-sounding engineering outputs without possessing genuine engineering rigor**. The output may appear competent to a non-expert reader, but it lacks:

| Attribute | Prompt Engineering | Domain Reasoning Compiler |
|-----------|-------------------|---------------------------|
| **Explicit reasoning path** | ❌ Hidden in model weights | ✅ Represented as PDG |
| **Dependency tracking** | ❌ Implicit | ✅ Formal graph edges |
| **Validation integration** | ❌ Post-hoc, optional | ✅ Built into compilation |
| **Evidence traceability** | ❌ Not guaranteed | ✅ Required at each node |
| **Reusability** | ❌ Tied to specific prompt | ✅ Compiled templates |
| **Inspectability** | ❌ Black box | ✅ Fully transparent |
| **Composability** | ❌ Monolithic | ✅ Modular primitives |

Traffic Agentic Engineering therefore requires a different approach: **reasoning must become explicit, structured, and executable** — not merely suggested through natural language instructions.

---

### 11.3 The Compilation Target Is Not Code

To understand the Domain Reasoning Compiler, it is helpful to draw an analogy with traditional software compilers — while also highlighting the crucial differences.

A traditional software compiler (such as GCC for C or javac for Java) performs the following transformation:

```
Source Code (High-Level Language)
    ↓ [Lexical Analysis]
    ↓ [Parsing]
    ↓ [Semantic Analysis]
    ↓ [Intermediate Representation]
    ↓ [Code Generation]
    ↓ [Optimization]
Machine Code (Executable Instructions)
```

The compilation target is **machine code** — binary instructions that a processor can directly execute. The compiler's job is to translate human-readable programming syntax into machine-executable operations.

The Domain Reasoning Compiler operates on a fundamentally different plane:

```
Engineering Responsibility
    ↓ [Responsibility Parsing]
    ↓ [Strategy Selection]
    ↓ [Primitive Selection]
    ↓ [Dependency Graph Construction]
    ↓ [Reasoning Trace Generation]
    ↓ [Validation Integration]
Executable Engineering Reasoning Graph
```

The compilation target is **not machine code**. It is an **executable engineering reasoning graph** — a structured representation of how engineering knowledge should be applied to fulfill a specific responsibility.

#### Key Differences from Traditional Compilers

| Aspect | Software Compiler | Domain Reasoning Compiler |
|--------|------------------|---------------------------|
| **Source language** | Programming language (C, Python, etc.) | Engineering responsibility (natural language) |
| **Target** | Machine code / bytecode | Executable reasoning graph (PDG) |
| **Syntax** | Formal grammar defined by language spec | Informal, domain-specific patterns |
| **Semantics** | Well-defined operational semantics | Context-dependent engineering meaning |
| **Optimization** | Register allocation, loop unrolling | Primitive reuse, evidence sharing, parallel execution |
| **Error handling** | Syntax errors, type errors | Missing constraints, invalid dependencies, evidence gaps |
| **Output consumer** | CPU / runtime environment | Traffic Agent Runtime (Chapter 13) |
| **Verification** | Type checking, static analysis | Constraint validation, simulation verification |

The DRC does not generate programs in the traditional sense. It generates **engineering reasoning workflows** — structured processes that specify exactly how engineering knowledge should be executed to transform an engineering responsibility into an engineering deliverable.

---

### 11.4 The Source Language of DRC: Engineering Responsibility

The source language of the Domain Reasoning Compiler is **engineering responsibility** expressed in natural language.

This is both the most intuitive and the most challenging aspect of the DRC design. It is intuitive because engineers naturally express their work as responsibilities:

> *"Optimize the signal timing of Intersection A."*
>
> *"Diagnose the cause of recurring congestion on Main Street corridor."*
>
> *"Evaluate whether the proposed lane reconfiguration will improve LOS."*
>
> *"Predict traffic demand growth for the next five years."*

It is challenging because these expressions contain **no explicit algorithms**, **no specified methods**, and **no declared dependencies**. The compiler must infer all of these from the responsibility statement itself.

#### What the Compiler Must Infer

Consider the responsibility: *"Optimize the signal timing of Intersection A."*

From this simple statement, the DRC must infer:

| Inference Category | What Must Be Determined |
|--------------------|------------------------|
| **Analysis Strategy** | This is a Design strategy (Chapter 10), not Diagnosis, Evaluation, or Prediction |
| **Required Primitives** | Capacity analysis, cycle length calculation, split optimization, pedestrian clearance, coordination check |
| **Engineering Constraints** | Minimum/maximum green times, pedestrian intervals, phase sequences, hardware limits |
| **Evidence Requirements** | Traffic volumes, geometry, existing timing, pedestrian counts, sight distances |
| **Validation Procedures** | Constraint checking, HCM LOS calculation, simulation verification |
| **Deliverable Format** | Signal timing plan + engineering report with before/after comparison |

None of this information is explicitly stated in the original responsibility. Yet all of it is **implicitly required** for the responsibility to be properly fulfilled.

#### The Compilation Challenge

This inference problem represents the core challenge of the DRC:

> **How can a system transform an underspecified natural-language intent into a fully specified, executable engineering workflow?**

The answer lies in the layered architecture of TAE itself:

1. **Responsibility Parsing** extracts the semantic structure of the request
2. **Strategy Selection** maps the parsed responsibility to one of the four fundamental strategies (Chapter 10)
3. **Primitive Selection** identifies which engineering primitives from the primitive library (Chapters 5-7) are applicable
4. **Dependency Graph Construction** assembles the selected primitives into a valid execution order (PDG)
5. **Reasoning Trace Generation** instantiates the abstract graph with concrete data and parameters
6. **Delivery Assembly** produces the final engineering deliverable in the required format

Each layer resolves additional ambiguity, until the originally vague responsibility becomes a precise, executable process.

---

### 11.5 The Intermediate Representation: Primitive Dependency Graph

Modern software compilers translate source programs through one or more **intermediate representations (IRs)** — abstract formats that capture the program's semantics while being independent of both the source language and the target architecture. Common IRs include:

- **Abstract Syntax Tree (AST):** Captures syntactic structure
- **Three-Address Code:** Captures low-level operations
- **Static Single Assignment (SSA) Form:** Enables optimization passes
- **Control Flow Graph (CFG):** Captures execution paths

Traffic Agentic Engineering adopts the same philosophical approach but with a domain-specific intermediate representation: the **Primitive Dependency Graph (PDG)**, introduced in detail in Chapter 7.

#### PDG vs. Traditional IR

| Characteristic | AST / CFG (Traditional IR) | PDG (TAE IR) |
|---------------|---------------------------|--------------|
| **Nodes represent** | Program statements / basic blocks | Engineering primitives |
| **Edges represent** | Control flow / data flow | Engineering dependencies |
| **Captures** | Program syntax and execution logic | Engineering reasoning structure |
| **Determines** | Execution order of operations | Application sequence of primitives |
| **Enables** | Code optimization | Reasoning optimization, validation insertion |
| **Validates** | Type safety, reachability | Completeness, constraint satisfaction |

Unlike an AST, which represents the **syntax** of a program, the PDG represents the **reasoning** of an engineering process. Each node corresponds to an engineering primitive — a well-defined unit of engineering computation. Each edge represents an engineering dependency — the requirement that one primitive's output serves as another's input, or that one primitive must execute before another due to logical precedence.

#### Example PDG: Signal Timing Optimization

Consider again the responsibility: *"Optimize the signal timing of Intersection A."*

The DRC would generate a PDG approximately as follows:

```
                    ┌─────────────────┐
                    │  Responsibility │
                    │   Parser        │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Strategy Select │
                    │  → Design       │
                    └────────┬────────┘
                             │
                             ▼
              ┌──────────────────────────────┐
              │     Primitive Selection      │
              └──────────┬───────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
  ┌────────────┐ ┌────────────┐ ┌────────────┐
  │  Traffic   │ │ Geometric  │ │ Existing   │
  │  Volume    │ │  Data      │ │  Timing    │
  │  Primitive │ │  Primitive │ │  Primitive │
  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘
        │              │              │
        ▼              ▼              ▼
  ┌─────────────────────────────────────────┐
  │          Capacity Analysis              │
  │             Primitive                   │
  └──────────────────┬──────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────┐
  │       Cycle Length Calculation           │
  │             Primitive                    │
  └──────────────────┬──────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────┐
  │         Split Optimization               │
  │             Primitive                    │
  └──────────────────┬──────────────────────┘
                     │
            ┌────────┴────────┐
            ▼                 ▼
  ┌─────────────────┐ ┌─────────────────┐
  │  Constraint     │ │  Pedestrian     │
  │  Validation     │ │  Clearance      │
  │  Primitive      │ │  Check          │
  └────────┬────────┘ └────────┬────────┘
           │                   │
           ▼                   ▼
  ┌─────────────────────────────────────────┐
  │        Simulation Validation             │
  │             Primitive                    │
  └──────────────────┬──────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────────┐
  │     Deliverable Assembly                │
  │   (Timing Plan + Report)                │
  └─────────────────────────────────────────┘
```

This graph captures not just *what* computations are needed, but *how they relate* to each other — which is precisely the information required for systematic execution, validation, and delivery.

---

### 11.6 The Complete Compiler Pipeline

The Domain Reasoning Compiler operates as a multi-stage pipeline, transforming engineering responsibility through progressively more structured representations until it produces an executable engineering process.

#### Stage 1: Responsibility Parsing

**Input:** Natural language engineering responsibility  
**Output:** Structured responsibility representation

The parser extracts the semantic components of the responsibility statement:

| Component | Example Extraction |
|-----------|-------------------|
| **Action verb** | "Optimize" → Design action |
| **Target object** | "signal timing" → Signal timing domain |
| **Scope** | "Intersection A" → Specific location |
| **Implicit goals** | Improve efficiency, reduce delay, maintain safety |
| **Constraints hints** | None explicit → apply defaults |

The parser must handle the inherent ambiguity of natural language. Different engineers might express the same responsibility in different ways:

> *"Fix the timing at Intersection A."*
> *"Make Intersection A work better."*
> *"The signals at A need retiming."*

All three should resolve to the same parsed representation.

#### Stage 2: Strategy Selection

**Input:** Parsed responsibility  
**Output:** Selected analysis strategy (Diagnosis / Design / Evaluation / Prediction)

Using the framework established in Chapter 10, the compiler maps the parsed responsibility to one of the four fundamental strategies. This mapping is determined by:

- The **action type** implied by the responsibility verb
- The **goal orientation** (find cause vs. create solution vs. assess performance vs. estimate future)
- **Contextual cues** (is there an existing problem to diagnose? a new design to create?)

#### Stage 3: Primitive Selection

**Input:** Selected strategy + parsed responsibility  
**Output:** Set of applicable engineering primitives

From the primitive library (Chapters 5-7), the compiler selects all primitives that are relevant to the selected strategy and responsibility scope. Selection criteria include:

- **Domain relevance:** Does this primitive operate on the target domain (signals, geometry, demand)?
- **Strategy compatibility:** Is this primitive typically used within the selected strategy?
- **Scope appropriateness:** Does this primitive operate at the right level of detail?

#### Stage 4: Primitive Dependency Graph Construction

**Input:** Selected primitives  
**Output:** Ordered PDG with explicit dependencies

This is the core compilation stage. The selected primitives are assembled into a dependency graph that specifies:

- **Data dependencies:** Which primitive outputs serve as inputs to other primitives?
- **Order dependencies:** Which primitives must complete before others can begin?
- **Validation dependencies:** Where do validation checks need to be inserted?
- **Evidence dependencies:** Which primitives require external evidence (observations, measurements)?

The resulting PDG serves as the **intermediate representation** — a complete, formal specification of the engineering reasoning process.

#### Stage 5: Reasoning Trace Generation

**Input:** PDG + collected evidence  
**Output:** Instantiated reasoning trace with concrete values

The abstract PDG is instantiated with actual data:

- Traffic counts replace abstract "volume" inputs
- Geometric measurements replace abstract "geometry" parameters
- Existing timing plans replace abstract "current state" references
- Calculated values propagate through dependency edges

At this stage, the reasoning process transitions from **template** to **instance** — from a general engineering method to a specific application to a specific engineering problem.

#### Stage 6: Delivery Assembly

**Input:** Completed reasoning trace + validation results  
**Output:** Engineering deliverable

The final stage assembles all intermediate results, validation outcomes, and evidence references into a structured engineering deliverable — a signal timing plan, diagnosis report, assessment document, or forecast report, depending on the selected strategy.

#### Pipeline Summary

| Stage | Input | Output | Transformation |
|-------|-------|--------|----------------|
| **1. Parse** | Natural language responsibility | Structured representation | Text → Structure |
| **2. Strategy** | Parsed responsibility | Analysis strategy | Intent → Category |
| **3. Select** | Strategy + responsibility | Primitive set | Scope → Components |
| **4. PDG** | Primitive set | Dependency graph | Components → Workflow |
| **5. Trace** | PDG + Evidence | Instantiated trace | Template → Instance |
| **6. Deliver** | Trace + Validation | Engineering artifact | Process → Product |

By the final stage, engineering knowledge — which began as descriptive text in manuals and specifications — has been transformed into an **executable engineering process** that produces a validated, evidence-backed deliverable.

---

### 11.7 DRC Is the Missing Layer of Agent Engineering

To appreciate the role of the Domain Reasoning Compiler, it is necessary to situate it within the broader landscape of AI agent frameworks.

Current agent frameworks — including LangChain, AutoGPT, CrewAI, and numerous others — focus primarily on **execution capabilities**:

| Capability | Description | Framework Support |
|------------|-------------|-------------------|
| **Tool invocation** | Calling external APIs, databases, functions | ✅ Mature |
| **Workflow orchestration** | Sequencing multi-step tasks | ✅ Mature |
| **Memory management** | Maintaining conversation context, long-term storage | ✅ Mature |
| **Multi-agent collaboration** | Coordinating specialized agents | ✅ Emerging |
| **Reasoning construction** | Building domain-specific推理逻辑 from knowledge | ❌ **Missing** |

These frameworks excel at answering the question: **"How does an agent execute?"**

They do not adequately address the question: **"How is engineering reasoning constructed in the first place?"**

#### The Gap Between Execution and Reasoning

Consider the distinction:

- **Execution** is the mechanics of running a process — invoking tools, passing data, handling errors, managing state. Current frameworks handle this well.
- **Reasoning construction** is the engineering of determining *what* process should run, *in what order*, *with what dependencies*, *using what domain knowledge*, and *producing what validations*. This is where current frameworks fall short.

The Domain Reasoning Compiler fills precisely this gap. It is the component that **constructs** the reasoning process that agent frameworks then **execute**.

#### Without DRC: Agent as Workflow Engine

An agent system without a Domain Reasoning Compiler reduces to a sophisticated workflow engine:

- It can invoke tools in sequence
- It can maintain state across interactions
- It can follow pre-defined procedures
- But it **cannot construct new engineering reasoning processes** from engineering knowledge

The engineering logic must be hard-coded by developers, embedded in prompts, or manually configured — none of which scales to the breadth and depth of real traffic engineering.

#### With DRC: Agent as Engineering System

An agent system equipped with a Domain Reasoning Compiler becomes a genuine **engineering system**:

- It can accept engineering responsibilities expressed in natural language
- It can construct appropriate reasoning processes by compiling engineering knowledge
- It can validate intermediate and final results against engineering standards
- It can produce structured, evidence-backed engineering deliverables

The transition is fundamental: **from an agent that executes predefined workflows to an agent that engineers solutions from first principles of domain knowledge**.

---

### 11.8 From Traffic Engineering to Traffic Agentic Engineering

The introduction of the Domain Reasoning Compiler marks a conceptual watershed in the evolution of traffic engineering as a discipline.

#### Traditional Traffic Engineering: Knowledge as Documents

Traditional traffic engineering organizes its knowledge in **human-centric forms**:

| Knowledge Type | Traditional Form | Accessibility |
|---------------|-----------------|---------------|
| Methodologies | Manual chapters, textbook sections | Human-readable only |
| Calculation procedures | Step-by-step descriptions | Requires human interpretation |
| Design guidelines | Specification documents | Human judgment required |
| Decision heuristics | Experienced engineer's intuition | Tacit, personal |
| Validation criteria | Professional standards | Human expertise needed |

In this paradigm, **every act of engineering requires a human engineer** to:
1. Read and understand the relevant knowledge documents
2. Interpret which methodologies apply to the specific situation
3. Execute calculations and analyses manually or with tools
4. Apply professional judgment to interpret results
5. Produce deliverables in appropriate formats

The knowledge is **documented but not executable**. It exists passively in books and manuals, waiting for human intelligence to activate it.

#### Traffic Agentic Engineering: Knowledge as Executable Structures

Traffic Agentic Engineering reorganizes the same knowledge into **machine-executable forms**:

| Knowledge Type | TAE Form | Executability |
|---------------|---------|---------------|
| Methodologies | Compiled reasoning templates | Automatically selectable and executable |
| Calculation procedures | Engineering primitives with typed interfaces | Composable, reusable, auditable |
| Design guidelines | Constraint primitives in PDG | Enforced during compilation |
| Decision heuristics | Strategy selection rules | Systematically applied |
| Validation criteria | Validation primitives integrated in PDG | Automatically checked |

In this paradigm, engineering knowledge becomes **active rather than passive**:

1. The DRC **parses** the engineering responsibility
2. The DRC **selects** the appropriate strategy and primitives
3. The DRC **constructs** the reasoning graph
4. The DRC **executes** the reasoning process with real data
5. The DRC **validates** results against engineering standards
6. The DRC **assembles** the final deliverable

The transition represents a fundamental shift: **from human-centered engineering to agent-centered engineering**.

#### What Changes and What Stays the Same

| Aspect | Traditional Engineering | Traffic Agentic Engineering |
|--------|----------------------|---------------------------|
| **Engineering knowledge validity** | Unchanged — same HCM, same MUTCD, same theory | Same foundations |
| **Role of professional judgment** | Transferred to system design and oversight | Shifted upstream |
| **Calculation accuracy** | Potentially improved — reduced human error | Automated, reproducible |
| **Scalability** | Limited by number of engineers | Limited by computational resources |
| **Consistency** | Varies by engineer | Standardized via compiled templates |
| **Auditability** | Depends on individual documentation | Built into reasoning traces |
| **Accessibility** | Requires years of training | Available on demand |
| **Innovation** | Driven by individual creativity | Driven by template improvement + human oversight |

#### The Long-term Vision

The ultimate vision of Traffic Agentic Engineering is that **engineering knowledge is no longer merely documented — it becomes executable, verifiable, and continuously reusable**.

Every completed engineering responsibility:
- Produces a validated deliverable
- Generates a reusable reasoning template
- Accumulates evidence for organizational learning
- Improves the compiler's future performance

This creates a **virtuous cycle**: more engineered solutions → more accumulated experience → better compiled reasoning → more engineered solutions.

---

### Chapter Summary

The Domain Reasoning Compiler is the theoretical and practical bridge between **traditional traffic engineering** and **traffic agentic engineering**. Its key contributions are:

1. **Identification of the executability gap:** Engineering knowledge exists in descriptive forms that machines cannot directly execute.

2. **Demonstration of prompt engineering limits:** Natural language prompts cannot capture the explicit dependencies, validation requirements, and evidence structures that engineering reasoning demands.

3. **Definition of a novel compilation model:** The DRC compiles engineering responsibility (source language) into executable reasoning graphs (target), using the Primitive Dependency Graph as intermediate representation.

4. **Specification of a six-stage pipeline:** Responsibility Parsing → Strategy Selection → Primitive Selection → PDG Construction → Reasoning Trace Generation → Delivery Assembly.

5. **Positioning within agent architectures:** The DRC fills the missing layer between general-purpose agent execution frameworks and domain-specific engineering reasoning.

6. **Conceptual framing of the paradigm shift:** From human-centered, document-based engineering to agent-centered, executable engineering.

The DRC ensures that engineering knowledge — accumulated over decades in manuals, textbooks, specifications, and professional experience — can finally transcend its documentary form and become **genuine engineering intelligence**: systematic, scalable, verifiable, and continuously improving.

The next chapter examines the linguistic foundation that makes this compilation possible: the **Engineering Responsibility Language (ERL)**.

---
