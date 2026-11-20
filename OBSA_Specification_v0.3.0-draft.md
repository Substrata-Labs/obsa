# OBSA: Obligation-Based Systems Assurance

## Core Specification

**Version 0.3.0-draft**
**Status: Working Draft**
**Substrata Labs LLC**
**November 2026**

-----

## 1. Introduction and Scope

### 1.1 Purpose

This document specifies OBSA (Obligation-Based Systems Assurance), an ontology for compositional quality assurance in complex systems. OBSA defines a typed declaration hierarchy, structural invariants, composition rules, and conformance criteria that enable automated derivation of verification artifacts, compositional evidence accumulation, and computed traceability.

### 1.2 Novel Contribution

OBSA formalizes the *obligation* as an independent computational artifact occupying the layer between stakeholder requirements and engineering designs. The obligation is both design-independent (it does not prescribe implementation) and directly testable (it can be verified without knowing which design satisfies it). This dual property makes the obligation the correct pivot for automated derivation. The individual concepts (requirements decomposition, contract-based design, verification hierarchies) are established in systems engineering literature. Their organization into a typed, computable hierarchy with formal refinement, compositional evidence, and computed gap analysis is new.

### 1.3 Domain Neutrality

OBSA applies to any domain where components are composed through interfaces and quality depends on constraints that cross boundaries: aerospace vehicles, electronic subsystems, embedded firmware, software services, manufacturing processes, field operations, cloud infrastructure, and fleet management. The declaration hierarchy, structural rules, composition rules, and conformance criteria are domain-invariant. Domain-specific concerns are addressed through harnesses and adapters, which are implementation details outside the scope of this specification.

### 1.4 Declaration Sets and Configurations

A declaration set represents one system configuration at one point in time. This is a property of a declaration set, not a limit on OBSA: systems with multiple configurations are assured by composing or comparing declaration sets (Section 8.5). Fleet variants, concurrent software versions, hardware revisions, and multi-configuration analysis are handled by composing or comparing multiple declaration sets, not by extending a single one. Section 8.5 defines the rules for composition across declaration sets. Multi-configuration management (tracking which declarations differ between configurations and propagating changes) is an implementation concern outside this specification.

### 1.5 What This Specification Does Not Define

This specification defines the abstract data model and the rules governing it. It does not define:

- Computational mechanisms, evaluation engines, or algorithms (but see Section 1.6 for required properties of any conforming evaluation engine)
- File formats, serialization schemes, or wire protocols
- Tooling, user interfaces, or development environments as implementations. The qualification status of any tool that produces or judges evidence (evaluation engine, harness, adapter, run planner) is in scope (Rule 12.2.5).
- Organizational processes, approval workflows, or meeting structures
- Harness or adapter implementations

The specification is the shared language. The computational mechanism is a separate concern.

### 1.6 Evaluation Engine Requirements

This specification defines the declaration model and the rules governing it. It does not define the evaluation engine that checks conformance, computes gaps, and produces verdicts. However, the rules in this specification assume an evaluation engine with the following properties. Any conforming implementation MUST satisfy these requirements.

**Bounded evaluation.** Every structural rule check, refinement computation, composition rule check, and gap analysis computation MUST complete within a bounded number of operations for any valid declaration set. The specific bound is an implementation parameter. The existence of a bound is an architectural requirement. If a computation exceeds the bound, the engine MUST return a definite failure status (not silently hang or produce partial results). A timeout on a specific check is treated as an inconclusive result: the affected verification conditions remain in the demanded set and are bucketed `undecidable` unless an earlier bucket of Rule 9.3.4 applies, the obligation's verdict is no better than `incomplete` (Section 4.7.3), and the check is reported as requiring alternative evidence (typically analysis).

**Topological evaluation.** The engine MUST evaluate the declaration dependency graph in an order consistent with the graph's topological structure, processing every node exactly once. If a cycle is detected in any reference graph (parent references, children references, satisfies references, decomposed_into references), the engine MUST halt evaluation and report a structural error before any conformance assessment is produced. Every declared constraint receives a definite status: satisfied, violated, or incomplete. These statuses coarsen the per-condition coverage buckets of Rule 9.3.4 (Section 4.7.3). No node is skipped. No dependency is left unresolved within the current evaluation scope.

**Deterministic evaluation.** The engine MUST produce identical conformance reports, gap analyses, and verdicts for identical inputs (declaration set plus evidence set) regardless of evaluation time, evaluation order among independent nodes, the platform executing the engine (not a verification environment), or number of evaluation passes. No evaluation-time state, no external dependencies during evaluation, no non-deterministic operations. Identical inputs, identical outputs. Any external corpus the evaluation reads (for example a requirements export used for `record_id` resolution or coverage scoping) is a pinned, versioned input and part of 'identical inputs'; no live lookup occurs during evaluation.

**Gated execution.** The engine MUST NOT produce deployment or release artifacts from a declaration set that fails Level 1 structural conformance (Section 12.1). Any gated artifact MUST carry the conformance report's tool-qualification and admission-basis statements (Rules 12.2.5, 10.2.6). The execution function (whatever operation the organization gates on conformance) is defined only when validation passes. The specific gate (CI pipeline check, release approval, deployment authorization) is an organizational concern outside this specification. The requirement that a gate EXISTS and that it is conditioned on conformance is within scope.

**Verdict authority and self-certification.** Verdicts are computed by the engine from measurements (Rule 9.3.2), not taken from the producing tool. The engine is itself a tool subject to qualification; its own outputs never constitute its qualification (Rule 12.2.5).

-----

## 2. Terms and Definitions

Terms are listed alphabetically. Where a term is formally defined as an instance type (Section 4), the definition here is a summary; the normative definition is the instance type schema.

**Acceptance function.** A function defined by a constraint type that evaluates an evidence payload and returns a verdict. The acceptance function determines whether measured, observed, or assessed values satisfy the constraint's assertion.

**Adapter.** A template that renders abstract verification artifacts into domain-specific executables, procedures, or inspection criteria. One adapter per target tool or environment. An adapter is a tool; see Rule 12.2.5.

**Budget (design shape).** A design shape modeling a finite resource with supply, demand, allocation, and capacity constraints.

**Co-execution.** The discharge of two or more verification procedures from a single verification run.

**Compatibility (design shape).** A design shape modeling agreement on versions, parameters, tolerances, or specifications across a boundary between components.

**Component.** A discrete element that performs work or contributes a property. The atomic element of system decomposition. Has intrinsic properties independent of its connections. Formally defined as an instance type in Section 4.6.

**Composition.** The process by which independently verified child obligations combine to satisfy a parent obligation.

**Conformance.** The property of a declaration set satisfying the structural rules and, optionally, the evidence completeness criteria defined in this specification.

**Constraint.** A specific, parameterized assertion that an obligation requires to hold. Typed by one of the recognized constraint types.

**Constraint type.** A category of assertion pattern with a defined refinement rule and acceptance function. See Section 6.

**Coverage.** The fraction of demanded verification conditions that are discharged, computed against a declared coverage query over a pinned corpus version (Rules 9.3.1, 9.3.5).

**Coverage bucket.** One of the mutually exclusive classes into which every demanded verification condition is placed (Rule 9.3.4).

**Coverage query.** A named selection of Requirements, and with them their active descendant obligations, together with the pinned corpus version it is evaluated against (Rule 9.3.5).

**Decomposition gap.** A gap where a requirement exists but has not been decomposed into obligations.

**Derivation.** Whether a verification procedure was generated through a harness and adapter (`derived`) or references an existing procedure (`authored`). Formerly named `provenance` (Section 4.7.1).

**Design.** The physical realization of one or more obligations. Binds obligations to specific components, materials, protocols, parameters, and constraints. Instantiated as one of the recognized design shapes. The locus of evidence production.

**Design shape.** A category of design determined by the kind of relationship it models. See Section 5.

**Element.** A declared Component (Section 4.6) named by an obligation as something the claim is about. Components are atomic; a claim about an assembly names its constituent components. Scope class is determined by the number of named elements and attribution (Section 4.2).

**Envelope (design shape).** A design shape modeling the boundary of validated capability, operating range, or characterization evidence across one or more dimensions.

**Evidence.** An artifact produced by verification execution, inspection, analysis, or demonstration that contains a structured measurement payload evaluable by a constraint type's acceptance function. Evidence is produced at and attached to the design level.

**Exchange (design shape).** A design shape modeling structured transfer of data, energy, material, or force between components.

**Gap.** A structural deficiency in the declaration set where an obligation lacks required evidence. Gap types are enumerated in Section 9.1.

**Harness.** A mapping from constraint types to abstract verification artifacts for a specific interface medium or verification domain. A harness is a tool; see Rule 12.2.5. Distinct from the steerable control-and-observation sense of a verification environment (Section 4.5).

**Infrastructure gap.** A gap where an obligation exists but no harness can derive verification artifacts for its medium and no authored verification procedure covers the required evidence.

**Interaction constraint.** A constraint on the joint relationship between two or more dimensions of an n-dimensional envelope, expressing a coupling that cannot be captured by independent per-dimension ranges alone.

**Interface.** The coupling between exactly two elements. Not a declared instance type. The pairwise interface view (Section 4.7.7) lists designs whose components include both elements. A claim whose value depends on a shared bus or network is not an interface claim (Section 4.2).

**Interference.** A relation between two verification procedures such that co-executing them in one run would invalidate the evidence of at least one. Computed, not authored. See Section 7.12.

**Measurement footprint.** The declared properties of a verification plan entry (Section 4.8.2) that determine how it co-executes with others: whether it injects a stimulus or observes passively, which operating dimensions its maneuver drives, and which channels it observes. See Section 4.8.2.

**Mode (design shape).** A design shape modeling valid states, transitions, guards, and invariants for a stateful element.

**Monotonic gap closure.** The property that the set of declared obligations only grows (never shrinks; an obligation leaves the active set only through a recorded status transition under Rule 8.4.1) and the evidence base only grows, so coverage (Rule 9.3.5) cannot be raised by editing designs, design verification plans, harnesses or adapters, because its denominator does not change under those edits. It changes only when evidence is produced or when an obligation-side declaration changes (an obligation withdrawn under Rule 8.4.1, `applicable_environments`, plan-entry environment bindings, or `operating_point_selection`). Obligation-side changes are reviewable declarations, and the report lists them against the previous corpus version.

**Obligation.** A testable, design-independent constraint allocated to a scope and level. It names the elements it is about (Section 4.2) and is reachable from a Requirement with a named external source (Rule 7.1.5). The pivot of the OBSA architecture. See Section 4.2.

**Observability gap.** A gap where an obligation exists but no telemetry mapping can produce runtime measurements for it.

**Operating condition.** A declared region of the operating state space that the system must tolerate or under which a verification is valid (Section 4.4).

**Operating dimension.** A named, typed axis of the operating state space, with a `kind` (exogenous, endogenous, or regime) and, for endogenous dimensions, declared dynamics. Formally defined in Section 4.4a.

**Operating point.** A set of an obligation's `applicable_conditions` that constrains each dimension those conditions constrain exactly once (Rule 7.7.5); the conjunction of the chosen conditions.

**Operating state space.** The product S = D₁ × … × Dₙ of all declared operating dimensions. A point in S is a complete operating regime of the system.

**Provenance.** The traceability of a requirement to a named source outside the verification artifacts (Rule 7.1.5). Not the derivation of a verification procedure.

**Refinement.** The relationship between a design and an obligation where the design's constraints are at least as strict as the obligation's constraints. Refinement establishes that evidence satisfying the design's constraints necessarily satisfies the obligation's constraints.

**Requirement.** A statement of need traced to a named external source of a declared kind (Section 4.1, Rule 7.1.5), identified, where recorded, in the requirements system of record by `record_id` (an unrecorded Requirement is a gap, Section 9.1). The top of the declaration hierarchy.

**Run covering.** A set of verification runs that collectively discharge every verification procedure in a target scope, each run feasible under Section 7.12. Computed by the evaluation engine.

**Scenario gap.** A gap where a field or production observation has no corresponding obligation.

**Source kind.** The category of a requirement's external origin (Section 4.1).

**System.** The composition of all components and interfaces. Has emergent properties: properties existing only among three or more elements and not attributable to any one element or any single pairwise interface.

**Tool.** Software that produces or judges evidence or computed values: the evaluation engine, harnesses, adapters, run planners, renderers. Subject to qualification (Rule 12.2.5).

**Traceability.** The computed property of the declaration dependency graph (DAG) that every requirement can be traced from its named external source through obligations and designs to evidence. Not maintained; derived.

**Trajectory.** A time-parameterized path through the operating state space traced by the system during a verification run.

**Verdict.** A computed assessment of whether an obligation is satisfied, derived from evidence attached to its satisfying designs and evaluated through the refinement relationship.

**Verification condition.** One (obligation, operating point, verification environment) triple demanded by an active obligation (Rule 9.3.1). What is counted is a thing to be established, not a test to be run.

**Verification environment.** The control and observation authority over the operating state space within which a verification run is realized: which operating dimensions it can set or drive (control authority), and which metrics it can measure (observation authority). A verification environment is not a passive context but a steerable, instrumented harness abstraction. A field environment is the degenerate case with zero control authority (all dimensions observed, not steered), where the verification environment reduces to a pure sensor over an externally imposed trajectory. Its capability (what it can drive and observe) is declared; whether its verdicts agree with reality (qualification) is a separate check (Rule 10.2.6). The normative definition is the instance type schema (Section 4.5).

**Verification method.** One of the four recognized methods for producing evidence: test, inspection, analysis, demonstration (TIAD). See Section 4.8.

**Verification plan.** A structured mapping from constraint parameters to verification methods, declared on both obligations and designs. Enables mechanical matching of evidence coverage.

**Verification run.** A single instantiation of a verification environment that follows one trajectory and produces one capture, from which evidence for one or more verification procedures is projected. Formally defined in Section 4.7.8.

-----

## 3. Architecture Overview

OBSA organizes declarations into three layers connected by typed references:

```
┌─────────────────────────────────────────────────────┐
│  REQUIREMENTS LAYER                                 │
│  Hazard analyses, FMEA, CAPA, product reqs,         │
│  interface contracts                                │
│  Language: "shall"                                  │
└──────────────────────┬──────────────────────────────┘
                       │ decomposes into
┌──────────────────────▼──────────────────────────────┐
│  OBLIGATION LAYER  (the pivot)                      │
│  Testable, design-independent constraints           │
│  Language: "must"                                   │
│  Properties: typed, scoped, leveled, testable       │
└──────────────────────┬──────────────────────────────┘
                       │ satisfied by
┌──────────────────────▼──────────────────────────────┐
│  DESIGN LAYER                                       │
│  Physical realizations in five shapes               │
│  Language: "how"                                    │
│  Properties: concrete, bound to components          │
│  Evidence attaches here                             │
└─────────────────────────────────────────────────────┘
```

Evidence flows upward through refinement:

```
Obligation declares: constraint + conditions + verification plan
    ↓ satisfied_by
Design declares: shape + components + stricter constraints + verification plan
    ↓ harness derives
VerificationProcedure: method-specific procedure per parameter
    ↓ execution produces
Evidence: structured measurement payload attached to design
    ↓ acceptance function evaluates
Design constraint satisfied → by refinement → Obligation constraint satisfied
    ↓ composition aggregates
Verdict: computed at obligation from design-level evidence
```

Traceability is a latent property of the dependency graph. Coverage and gap analysis are two projections of one partition of demanded verification conditions, computed in one pass (Section 9.3).

The evaluation of this architecture (checking structural rules, computing refinement, analyzing gaps, producing verdicts) requires an evaluation engine with specific properties: bounded computation, topological evaluation, determinism, gated execution, and verdict authority without self-certification. These properties are stated in Section 1.6. The specification defines the model. The engine evaluates it. Neither is sufficient alone.

### 3.1 Structural Parametricity

All computation defined by this specification, namely validation (Section 7), refinement (Rule 7.3), composition (Section 8), gap analysis (Section 9), traceability (Section 4.7.6), and verdict computation (Section 4.7.3), operates exclusively on type annotations and reference topology. No structural rule inspects, branches on, or depends on the domain meaning of content values.

This property is called *structural parametricity*. It is the architectural invariant that enables cross-domain composition (Rule 11.2.1) and prevents domain lock-in (Rule 11.2.2). It is not an accident of the current rule set; it is a necessary property derived from the infeasibility of building an evaluation engine that understands every possible domain.

**Litmus test.** Adding a new domain to OBSA requires zero changes to any structural rule, composition rule, conformance criterion, or gap analysis procedure. If a proposed structural rule would require modification when applied to a new domain, the rule violates structural parametricity and MUST be reformulated.

**Boundary clarification.** Structural computation MAY operate on content values through type-declared operations. Examples:

- CT-TIMING refinement applies ≤ to numeric values. The ≤ comparison is structural (defined by the constraint type). The number being compared is content (a domain-specific timing value).
- Interaction constraint expressions bind dimension names to declared dimensions. Name resolution is structural. What the dimension measures is content.
- Reference integrity checks (Rule 7.8.1) verify that referenced IDs exist. The existence check is structural. What the referenced component physically is in the real world is content.

These are structural operations: they use content values as opaque operands through typed interfaces. The evaluation function never *interprets* what those values mean in a domain. The distinction is: operating on content through a typed interface (structural) vs. understanding what content means (domain knowledge). The first is permitted. The second is excluded from structural computation.

**What is NOT structural.** The following are domain concerns, not structural computation:

- Whether a verification method is appropriate for a specific parameter
- Whether a component is the right physical entity for an obligation
- Whether a `maps_to` assertion on a VP entry is semantically valid
- Whether an interaction constraint expression models real physics
- Whether a design's components are sufficient for its obligation's scope, beyond including the elements the obligation names (Rule 7.2.4)
- Whether an obligation's claim is attributable to one element or to one pairwise interface, including whether its value depends on a shared bus or network (Section 4.2)
- Whether grounding references resolve to real artifacts

These are authored by domain experts at declaration time. The structural layer trusts these assertions and checks their form (references exist, types match, fields are present). Domain harnesses and domain tooling validate their substance; such tooling reports its qualification status (Rule 12.2.5).

-----

## 4. Instance Types

### 4.1 Requirement

A statement of need from an external source.

|Field           |Type              |Required|Description                                                                                                                       |
|----------------|------------------|--------|----------------------------------------------------------------------------------------------------------------------------------|
|id              |string            |yes     |Unique identifier. Format: `REQ-{domain}-{seq}` recommended.                                                                      |
|title           |string            |yes     |Human-readable name.                                                                                                              |
|source_type     |enum              |yes     |Source kind. One of: `hazard_analysis`, `fmea`, `capa`, `product_requirement` (includes use cases), `interface_contract`, `regulatory`, `customer`, `conops`, `manufacturing_standard`, `field_reliability`, `internal_standard`. `safety_analysis` is a deprecated alias of `hazard_analysis` (Annex C). A profile MAY restrict the admitted kinds (Annex D). Assurance priority is governed by Rule 7.1.7, not by this list.|
|source_reference|string            |yes     |Specific clause, analysis ID, or document reference identifying the named source. MUST NOT reference a verification procedure, evidence entry, test or verification plan (Rule 7.1.5).|
|record_id       |string            |no      |Identifier of this requirement in the requirements system of record. Absence is reported as an Unrecorded Requirement gap (Section 9.1), not a structural violation; a profile MAY require it (Annex D).|
|text            |string            |yes     |The requirement statement. Should use "shall" language.                                                                           |
|rationale       |string            |no      |Why this requirement exists.                                                                                                      |
|decomposed_into |list<ObligationID>|no      |Obligation IDs this requirement decomposes into. Computed field; see Rule 7.11.                                                  |

**Cardinality:** Many. Hundreds in a mature system.

**Example:**

```yaml
id: REQ-SAFE-001
title: Single Engine Continued Takeoff
source_type: hazard_analysis
source_reference: FHA-2025-003, Hazard H7; 14 CFR 25.121(a)
record_id: REQ-SAFE-001
text: "The aircraft shall maintain a positive climb gradient following
       a single engine failure recognized at or above V1 during takeoff."
decomposed_into: [OBL-SYS-OEI-CLIMB]
```

### 4.2 Obligation

A testable, design-independent constraint allocated to a scope and level.

|Field                  |Type                            |Required|Description                                                                                                                    |
|-----------------------|--------------------------------|--------|-------------------------------------------------------------------------------------------------------------------------------|
|id                     |string                          |yes     |Unique identifier. Format: `OBL-{level}-{name}` recommended.                                                                   |
|title                  |string                          |yes     |Human-readable name.                                                                                                           |
|parent                 |RequirementID or ObligationID   |yes     |Derivation parent: what this obligation was decomposed from.                                                                  |
|source_type            |enum                            |yes     |Origin of the obligation. One of: `requirement_decomposition`, `engineering_analysis`, `field_discovery`, `manufacturing_nonconformance`, `test_failure`. Obligations with origin `test_failure`, `field_discovery` or `manufacturing_nonconformance` MUST reach a Requirement with source kind `capa` or `fmea` (Rule 7.1.6).|
|scope                  |enum                            |yes     |One of: `intrinsic` (a property of exactly one element), `relational` (a property of the coupling between exactly two elements), `emergent` (a property existing only among three or more elements and not attributable to any one element or any single pairwise interface). See Rule 7.2.4. Scope is independent of `level`.|
|elements               |list<ComponentID>               |yes     |The elements this claim is about. Every ID MUST resolve (Rule 7.8.1). Count MUST match scope (Rule 7.2.4).                     |
|level                  |enum                            |yes     |One of: `system`, `subsystem`, `component`.                                                                                    |
|status                 |enum                            |yes     |One of: `active`, `superseded`, `withdrawn`. Default: `active`. See Rule 8.4.                                                  |
|constraint_type        |ConstraintTypeID                |yes     |Which constraint type (Section 6).                                                                                             |
|constraint             |Constraint                      |yes     |The specific parameterized assertion. See Section 6 for structure per type.                                                    |
|verification_plan      |list<VerificationPlanEntry>     |yes     |Mapping of constraint parameters to verification methods. At least one entry required. See Section 4.8.                        |
|applicable_conditions  |list<OperatingConditionID>      |yes     |Operating conditions under which this obligation requires evidence. No default; must be explicitly declared. Grouped into operating points per Rule 7.7.5, or restricted by `operating_point_selection` (Rule 7.7.6).|
|applicable_environments|list<VerificationEnvironmentID> |yes     |Verification environments in which this obligation demands evidence. Every listed environment is demanded (conjunctive). Authored at all levels; validated, never reduced, by capability derivation (Section 10.3).|
|operating_point_selection|{mode: full\|subset, points: list<list<OperatingConditionID>>, rationale: string}|no|Default full. When subset, Π(O) = points; rationale required (Rule 7.7.6).|
|provisional_closure    |{rationale, owner, date}        |no      |Records an accountable decision to treat the obligation as provisionally closed; see Rule 9.3.4.                               |
|rationale              |string                          |no      |Authored justification for the obligation: the non-attribution judgment for `emergent` scope (Rule 7.2.3), why each listed environment is credible for each quantity a safety obligation constrains (Rule 7.6.5), and the records required on withdrawal or relaxation (Rules 7.6.4, 8.4.1, 8.4.3).|

**Field classification:**

- **Permanent fields** (irreducible engineering judgment): `id`, `title`, `parent`, `source_type`, `scope`, `elements`, `level`, `status`, `constraint_type`, `constraint`, `verification_plan`, `applicable_conditions`, `applicable_environments`, `operating_point_selection`, `rationale`. Note: `applicable_conditions` is permanent because the relationship between operating condition parameters and obligation metrics requires domain knowledge that is not structurally derivable. Structured VE capabilities (Section 4.5) let the evaluation report conditions that a listed environment is not capable of exercising (Rule 10.3.1), but the conditions themselves remain authored.
- **Transitional fields:** none. (`applicable_environments` is permanent; structured capabilities validate it, Rule 10.3.1.) Deprecated fields are listed in Annex C.
- **Computed fields** (derived by evaluation engine, not authored): `children` (from scanning `parent` fields), `satisfied_by` (from scanning design `satisfies` fields). See Rule 7.11.

**Cardinality:** Dozens to hundreds. Obligations at level `system` are few (order 10). Obligations at level `subsystem` are moderate (order 10 per subsystem boundary). Obligations at level `component` are numerous (order 10 per component). `level` is the decomposition tier and is independent of scope class.

**Scope definitions:**

|Scope     |Applies to                                                                                                      |Example                                                                                                                                                                   |
|----------|----------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|intrinsic |A property of exactly one element                                                                               |Engine rated thrust, bolt material strength, function WCET                                                                                                                |
|relational|A property of the coupling between exactly two elements                                                         |FADEC channel-to-channel failure-detection latency over a dedicated cross-channel data link, bolt-pattern compatibility, API schema match between one producer and one consumer|
|emergent  |A property existing only among three or more elements, not attributable to one element or one pairwise interface|Continued takeoff climb gradient, bus timing on a shared avionics bus across its participants (Budget), total aircraft weight (Budget)                                    |

Any element may be verified by any method (Section 4.8.1). A claim whose value depends on a shared bus or network names its participants and is emergent; a claim about a dedicated link between two elements, or a medium-independent property of one producer-consumer pair, is relational. Additive totals over three or more elements are emergent and are typically satisfied by a Budget design (Section 5.5).

**Example:**

```yaml
id: OBL-SYS-OEI-CLIMB
title: One-Engine-Inoperative Climb Gradient
parent: REQ-SAFE-001
source_type: requirement_decomposition
scope: emergent
elements: [COMP-AIRFRAME, COMP-ENGINE-L, COMP-ENGINE-R, COMP-FLIGHT-COMPUTER]
level: system
status: active
constraint_type: CT-THRESHOLD
constraint:
  metric: oei_climb_gradient
  operator: ">="
  value: 2.4
  unit: percent
verification_plan:
  - parameter: oei_climb_gradient
    method: test
    rationale: "Flight test measurement under OEI conditions"
    environments: [VE-FLIGHT-TEST]
  - parameter: aerodynamic_model_correlation
    method: analysis
    rationale: "CFD model validated against wind tunnel and flight test data"
    environments: [VE-INTEGRATED-TEST]
  - parameter: pilot_engine_failure_recognition
    method: demonstration
    rationale: "Witnessed simulator evaluation per AC 25-7D"
    environments: [VE-INTEGRATED-TEST]
applicable_conditions:
  # three conditions on three different parameters: one operating point (Rule 7.7.5)
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-INTEGRATED-TEST
  - VE-FLIGHT-TEST
```

### 4.3 Design

The physical realization of one or more obligations. Every design is instantiated as one of the recognized shapes (Section 5). The design is the locus of evidence production: evidence is produced at and attached to the design level.

**Common fields (all shapes):**

|Field            |Type                       |Required   |Description                                                                                                                                                  |
|-----------------|---------------------------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
|id               |string                     |yes        |Unique identifier.                                                                                                                                           |
|shape            |enum                       |yes        |One of: `exchange`, `compatibility`, `envelope`, `mode`, `budget`.                                                                                           |
|title            |string                     |yes        |Human-readable name.                                                                                                                                         |
|satisfies        |list<ObligationID>         |yes        |At least one. Obligations this design satisfies.                                                                                                             |
|components       |list<ComponentID>          |yes        |At least one. References to Component instances (Section 4.6) involved in this design.                                                                       |
|constraints      |list<Constraint>           |yes        |Design-level constraints. Must refine obligation constraints per Rule 7.3.                                                                                   |
|verification_plan|list<VerificationPlanEntry>|yes        |Mapping of design parameters to verification methods. Must cover all obligation verification plan parameters for obligations in `satisfies`. See Section 4.8.|
|version          |string                     |no         |Version of this design declaration.                                                                                                                          |

**Cardinality:** Varies by shape. See Section 5.

### 4.4 OperatingCondition

A declared region of the operating state space that the system must tolerate or under which a verification is valid.

An OperatingCondition is expressed using the multi-dimensional region grammar of Section 5.3 (dimensions, ranges, and interaction constraints). Dimensions not named by the condition are unconstrained (the condition projects onto the named sub-space). A scalar OperatingCondition, a single `parameter` with a `range`, is the one-dimensional case (Rule 4.4.2).

|Field   |Type  |Required|Description                                                                                                                                                                                                                        |
|--------|------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|id      |string|yes     |Unique identifier.                                                                                                                                                                                                                 |
|title   |string|yes     |Human-readable name.                                                                                                                                                                                                               |
|region  |Region|yes     |A region in S. Scalar form `{dimension, range}` or multi-dimensional form `{dimensions, interaction_constraints}` per Section 5.3. Referenced dimension names MUST resolve to declared OperatingDimension instances (Section 4.4a).|
|severity|enum  |no      |One of: `nominal`, `degraded`, `extreme`.                                                                                                                                                                                          |

**Rule 4.4.1.** Every dimension name appearing in an OperatingCondition's `region` MUST correspond to a declared OperatingDimension (Section 4.4a). This is the reference-integrity check for conditions, analogous to Rule 7.8.1 for components. A dimension name used only by scalar-form conditions (Rule 4.4.2) and not declared is implicitly declared as an OperatingDimension with `kind: exogenous`, the condition's unit, and no dynamics, and a warning is reported. Multi-dimensional regions MUST reference explicitly declared dimensions.

**Rule 4.4.2 (scalar form).** An OperatingCondition MAY be written `parameter: <name>`, `range: <Range>`; this is equivalent to `region: {dimension: <name>, range: <Range>}`.

**Cardinality:** Typically 10-50 per system, each a region over one or more operating dimensions.

**Examples:**

```yaml
id: OC-MAX-PAYLOAD
title: Maximum Payload
region:
  dimension: payload_mass
  range: {min: 15000, max: 15000, unit: kg}
```

```yaml
id: OC-TAXI-MAX-LOADED
title: Loaded Taxi at Maximum Speed
region:
  dimensions:
    - {name: ground_speed, range: {min: 15, max: 15, unit: m/s}}
    - {name: payload_mass, range: {min: 15000, max: 15000, unit: kg}}
    - {name: surface_slope, range: {min: 0, max: 0.02, unit: rad}}
  interaction_constraints:
    - parameters: [ground_speed, surface_slope]
      expression: "ground_speed * surface_slope <= 0.25"
      rationale: "Validated envelope couples taxi speed and runway slope."
```

OC-MAX-PAYLOAD and OC-TAXI-MAX-LOADED share the payload dimension at a compatible value and are co-instantiable. A "Parked" condition pinning `ground_speed` to `{0,0}` is disjoint from OC-TAXI-MAX-LOADED on that dimension and is therefore instantaneously exclusive with it (Section 7.12).

### 4.4a OperatingDimension

A named, typed axis of the operating state space.

|Field   |Type        |Required   |Description                                                                                                                                    |
|--------|------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------|
|id      |string      |yes        |Unique identifier.                                                                                                                             |
|name    |string      |yes        |The dimension name referenced by conditions, envelopes, and footprints.                                                                        |
|unit    |string      |yes        |Unit of measurement (or `enum` for regime dimensions).                                                                                         |
|kind    |enum        |yes        |`exogenous`, `endogenous`, or `regime`. See below.                                                                                             |
|dynamics|Dynamics    |conditional|Required when `kind = endogenous`. Describes how a trajectory drives the dimension.                                                            |
|states  |list<string>|conditional|Required when `kind = regime`. The discrete values. A regime dimension MAY instead be realized by a Mode design (Section 5.4); see Rule 7.12.6.|

**Dimension kinds:**

- **exogenous**: Set at run setup and held for the run's duration by the environment (ambient temperature, payload mass, surface slope). Reversible: it can be reset for the next run. Freely combinable across dimensions.
- **endogenous**: Not set; driven by the trajectory as a function of its own history (actuator temperature, state of charge, accumulated wear, thermal soak). Typically monotone and only slowly or not at all reversible within a run. A dimension whose effective controllability in an environment is `driven` (Rule 4.5.1) is what converts instantaneous exclusivity into forced ordering in that environment (Section 7.12); `endogenous` is the kind from which `driven` is derived when the environment declares no control.
- **regime**: A discrete operating mode (flight mode, controller state). Exclusivity on a regime dimension is the discrete analogue of the state-dependent activation already defined in Rule 7.10.

**Dynamics (for endogenous dimensions):**

|Field                |Type   |Required|Description                                                                                                                                                                                                                                                               |
|---------------------|-------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|direction            |enum   |yes     |`increasing`, `decreasing`, or `bidirectional`. The direction a driving maneuver moves the dimension.                                                                                                                                                                     |
|reversible_within_run|boolean|yes     |Whether the dimension can be returned to a prior value within one run (e.g. thermal cooldown) or is one-way within a run (e.g. accumulated wear).                                                                                                                         |
|reset_between_runs   |boolean|yes     |Whether the dimension returns to a controlled value at the start of a new run in this environment. An environment MAY override it (Section 4.5). Whether a dimension behaves as held or driven in a given environment is its effective controllability there (Rule 4.5.1).|

**Endogenous dynamics.** `dynamics` is an authored annotation. At Level 2 and above, an endogenous dimension's dynamics SHOULD be backed by a model-validation obligation (Section 4.8.1), parented per Rule 7.1.6.

### 4.5 VerificationEnvironment

The context in which evidence is produced. Because a VerificationEnvironment carries control authority (which dimensions it can set or drive) and a run is a trajectory it realizes (Section 4.7.8), it is not a passive context: it is a **steerable, instrumented harness abstraction**: a control authority plus an observation authority over the operating state space. Note the terminology boundary: "Harness" in Section 2 is the constraint-to-artifact rendering mapping and is a distinct concept; the steerable-rig sense here is carried by the VE's control and observation authority, not by the term "harness." A field environment is the degenerate case: zero control authority, all dimensions `observed`, the VE reduced to a pure sensor over an externally imposed trajectory.

Structured capabilities (below) are optional at every conformance level. Where an environment declares them, they are validated (Rule 10.3.1). The unstructured `capabilities: list<string>` field remains valid.

|Field       |Type        |Required|Description                                           |
|------------|------------|--------|------------------------------------------------------|
|id          |string      |yes     |Unique identifier.                                    |
|title       |string      |yes     |Human-readable name.                                  |
|description |string      |yes     |What physical or simulated context this represents.   |
|realization |enum        |no      |One of: `physical`, `simulated`, `mixed`. Whether the environment observes the real system (or physical instrumentation of it), a model, or both. Descriptive; it defines no ordering (Section 10.1). Used by Rule 7.6.5.|
|capabilities|list<string>|no      |Unstructured description of what this environment can drive and observe. Valid at Level 1.|

**Informative note: structured capabilities.** Structured capabilities are specified below; the unstructured `capabilities` field remains valid at Level 1.

Environment names (for example a dynamometer cell, a simulation, an iron-bird rig, an integrated-system test, the field) are instance labels. OBSA defines no ordering among environments. A simulation may be the right environment for a claim where a hardware rig is not, and the reverse.

**Capability.** Capability is necessary, not sufficient, for an environment's evidence to be trusted; agreement of its verdicts with reality (qualification) is a separate check (Rule 10.2.6).

|Field                  |Type                  |Required|Description                                           |
|-----------------------|----------------------|--------|------------------------------------------------------|
|controllable_dimensions|list<DimensionControl>|no      |Which operating dimensions this environment can set and hold, and at what reversibility.|
|observable_metrics     |list<string>          |no      |The measurement metrics this environment can produce. Determines which evidence schemas it can satisfy. Matched lexically to constraint metrics (Rule 9.3.7).|
|cost                   |Cost                  |no      |Relative or absolute cost per run. Consumed by the run-covering computation (Rule 7.12.9) to trade window packing against run expense within this environment; it never removes or substitutes an environment an obligation lists (Rule 7.7.2) and is not used by any structural rule.|

**DimensionControl:**

|Field             |Type   |Required   |Description                                           |
|------------------|-------|-----------|------------------------------------------------------|
|dimension         |string |yes        |A declared OperatingDimension name.                   |
|controllability   |enum   |yes        |`settable` (the environment sets and holds it), `driven` (evolves per trajectory under the system's dynamics), or `observed` (not controlled, only measured, as in field).|
|mode              |enum   |conditional|For `settable` dimensions: `instantaneous` (the environment can set the value directly, e.g. a simulation initializing state) or `rate_limited` (the environment can command the value but must traverse to it through the system's own dynamics, e.g. a vehicle accelerating to a commanded speed). Required when `controllability = settable`. `driven` dimensions are always effectively rate_limited by their dynamics; `observed` dimensions have no control mode.|
|reset_between_runs|boolean|no         |Overrides the dimension's own `dynamics.reset_between_runs` for this environment.|

**Informative note.** `controllability` is per (dimension × environment), not intrinsic to the dimension. Actuator temperature is `driven` on hardware but `settable`/`instantaneous` in a full simulation that can initialize it; payload is `settable` when bolted on but `observed` in the field. This is the parameter that makes different environments points on one model (Annex B).

**Rule 4.5.1 (effective controllability).** The effective controllability of dimension d in environment E is E's DimensionControl for d, if declared; otherwise it is derived from d's `kind`: `exogenous` → `settable`/`rate_limited`; `endogenous` → `driven`; `regime` → `settable`/`rate_limited`. No default is `instantaneous`. Effective reset is: for `settable`, available unless E declares `reset_between_runs: false`; for `driven`, E's `reset_between_runs` if declared, else d's `dynamics.reset_between_runs`; for `observed`, none. Every computation in Section 7.12 uses effective controllability, never `kind` directly, except Rule 7.12.6 (regime dimensions realized by a Mode design). The kind-derived default is used only by Section 7.12; capability (Rule 10.3.1) counts only declared DimensionControls.

**Control mode and reachability.** `mode` captures steering authority, which is distinct from the ability to hold a region. A simulation sets any dimension `instantaneous`ly (state teleport), so any window order is reachable at negligible cost. A physical environment sets many dimensions `rate_limited`ly: it can command a target but must traverse to it through the system's real dynamics, which takes time and heats or depletes endogenous dimensions en route. This is why feasibility is a reachability question, not only a containment question (Rule 7.12.8), and why between-window traversal is a real cost in a physical run covering (Rule 7.12.9).

### 4.6 Component

A discrete element that performs work or contributes a property. The atomic element of system decomposition.

| Field     | Type   | Required | Description                                              |
|-----------|--------|----------|----------------------------------------------------------|
| id        | string | yes      | Unique identifier.                                       |
| title     | string | yes      | Human-readable name.                                     |
| source    | Source | yes      | Typed resolution to a real-world referent. See below.    |
| properties| map    | no       | Unstructured annotation map. See note below.             |

**Source:**

| Field              | Type   | Required | Description                                                     |
|--------------------|--------|----------|-----------------------------------------------------------------|
| resolution_protocol| string | yes      | How the reference resolves to a real-world artifact.            |
| reference          | string | yes      | The resolved identifier within the protocol's namespace.        |
| qualifier          | string | no       | Additional disambiguation (variant, version, configuration).    |

The `source` field is the grounding anchor. Its structure is domain-invariant: every component has a resolution protocol, a reference, and an optional qualifier. Its content is domain-specific: what constitutes a valid reference depends on the domain's artifact system. (The `source` field is the component's *grounding anchor* to a real-world referent; it is unrelated to a Requirement's source kind, Section 4.1. Its `qualifier` subfield disambiguates the reference and is unrelated to tool or environment qualification, Rule 12.2.5.)

The structural check is: does the component have a source with a non-empty `resolution_protocol` and a non-empty `reference`? The structural layer does not validate that the protocol is one of any predefined set; it validates that the fields are present and non-empty. Domain tooling validates that the protocol is meaningful and the reference resolves. This preserves structural parametricity: the evaluation function checks structure, never content.

**Recommended resolution protocols (informative):**

| Protocol             | Reference resolves to                  | Example                                              |
|----------------------|----------------------------------------|------------------------------------------------------|
| build_target         | A build system target                  | `//flight_controls/laws:pitch_law`                   |
| part_number          | A manufacturer or internal part number | `PW1133G-JM`                                         |
| model_library_node   | A node in a typed hardware model lib   | `model_library:nodes/fadec:eec_900_channel_a`        |
| file_path            | A file in a version-controlled repo    | `flight_controls/laws/pitch_law.cpp`                 |
| hardware_id          | A hardware identification scheme       | `CANopen:vendor_id=0x0000A1B2,product_code=0x0003`   |
| document_reference   | A document section or specification    | `EU-2023/1230:Article-10(6)`                         |
| drawing_number       | An engineering drawing or model number | `DWG-100-001`                                        |

New resolution protocols MAY be introduced for new domains without modifying the structural rules or schema.

The `properties` field is an unstructured annotation map. No structural rule references or validates its contents. It exists for domain tooling and human readability.

**Cardinality:** Dozens to hundreds. One per discrete element in the system.

**Examples:**

```yaml
id: COMP-SAFETY-RELAY
title: Emergency Stop Safety Relay
source:
  resolution_protocol: hardware_id
  reference: "CANopen:vendor_id=0x0000A1B2,product_code=0x0002"
  qualifier: "hw_rev:C"
properties:
  safe_inputs: 8
  safe_outputs: 4
  performance_level: "PL e"
```

```yaml
id: COMP-PITCH-CONTROL-LAW
title: Pitch Control Law Module
source:
  resolution_protocol: build_target
  reference: "//flight_controls/laws:pitch_law"
```

### 4.7 Derived and Semi-Derived Instance Types

These types are typically produced by computation. Except for VerificationRun (Section 4.7.8), their schemas are informative (implementations may vary in structure while preserving semantics).

VerificationProcedure may alternatively be authored directly to reference existing tests, inspections, analyses, or demonstrations that were not generated through the harness/adapter path. Authored procedures must satisfy the same schema and structural rules as derived procedures.

#### 4.7.1 VerificationProcedure

|Field                   |Type                     |Description                                                                                 |
|------------------------|-------------------------|--------------------------------------------------------------------------------------------|
|id                      |string                   |Unique identifier.                                                                          |
|derived_from_design     |DesignID                 |The design this procedure verifies.                                                         |
|obligation              |ObligationID             |The obligation this procedure ultimately addresses (via the design's `satisfies` reference).|
|parameter               |string                   |The specific constraint parameter under verification.                                       |
|method                  |enum                     |`test`, `inspection`, `analysis`, `demonstration`.                                          |
|operating_condition     |OperatingConditionID \| list<OperatingConditionID>|The operating point at which the procedure executes: the conditions of one of the obligation's operating points (Rule 7.7.5). A single ID is a one-condition point.|
|verification_environment|VerificationEnvironmentID|The environment at which the procedure runs. MUST be listed in the obligation's `applicable_environments` for its evidence to be admitted (Rule 10.2.5).|
|constraint_under_test   |Constraint               |The design-level constraint being checked.                                                  |
|evidence_schema         |EvidenceSchema           |Derived from `constraint_under_test`. Specifies the measurement payload that evidence produced by this procedure must contain. See Section 6.2 for per-constraint-type payload schemas.|
|procedure               |domain-specific          |The executable test, inspection checklist, analysis model, or demonstration protocol.       |
|derivation              |enum                     |`derived` (generated via harness/adapter) or `authored` (referencing existing procedure). Default: `derived`. `provenance` is a deprecated alias (Annex C). A `derived` procedure is a tool output (Rule 12.2.5).|

A VerificationProcedure inherits the measurement footprint of the plan entry it realizes (Section 4.8.2).

#### 4.7.2 Evidence

Evidence is produced at the design level. Obligation satisfaction is computed from design-level evidence through the refinement relationship.

|Field                   |Type                     |Description                                                                                |
|------------------------|-------------------------|-------------------------------------------------------------------------------------------|
|id                      |string                   |Unique identifier.                                                                         |
|verification_procedure  |VerificationProcedureID  |Which procedure produced this evidence.                                                    |
|design                  |DesignID                 |Which design this evidence is attached to.                                                 |
|parameter               |string                   |Which constraint parameter was measured, observed, or assessed.                            |
|method                  |enum                     |`test`, `inspection`, `analysis`, `demonstration`.                                         |
|operating_condition     |OperatingConditionID \| list<OperatingConditionID>|The operating point at which the evidence was produced.|
|verification_environment|VerificationEnvironmentID|At which environment.                                                                      |
|measurement             |Measurement              |Structured payload evaluable by the constraint type's acceptance function. See Section 6.2.|
|result                  |enum                     |`pass`, `fail`, `inconclusive`. Computed by the acceptance function. A supplied value is a claim; see Rule 9.3.2.|
|timestamp               |datetime                 |When evidence was produced.                                                                |
|artifact_reference      |string                   |Link to raw test result, inspection record, analysis report, demonstration log.            |
|run                     |VerificationRunID        |The verification run this evidence was projected from, when produced through the run-planning path.|
|component_revisions     |map<ComponentID, string> |Component IDs → revision identifiers at time of evidence production. See below.            |

**Component revision tracking.** Evidence is valid only for the component revisions it was produced against. The `component_revisions` field records the revision (commit hash, version tag, or other identifier) of every component touched by the design at the time the evidence was produced. When any component's revision changes, existing evidence for designs touching that component becomes *stale*: the evidence still exists in the record but no longer satisfies the obligation until re-verified against the current revisions. Evidence is current only if (a) its component revisions match and (b) it was judged against the constraint value current in the evaluated declaration set (Rule 9.3.2).

In monorepo configurations, all components may share a single revision identifier (the repository commit). In multi-repo configurations, each component carries its own revision. The evaluation engine (a tool, Rule 12.2.5) compares the `component_revisions` in evidence against the current declared component revisions to determine staleness.

**Projection note.** When evidence is produced through a verification run, multiple evidence entries MAY share a single `artifact_reference` (the run's capture) and a single `run`. Each entry still carries its own `parameter`, `measurement`, `operating_condition`, and `result`, recomputed independently (Rule 9.3.2) through its constraint type's acceptance function. Batching the execution does not batch the evaluation: one capture, many independent verdicts. This preserves the per-obligation granularity of Section 4.7.3.

**Evidence derivation.** When the procedure's `derivation` is `derived`, the evidence schema is also derived: the measurement structure is generated from the constraint, not authored. The generated test code produces evidence with the correct measurement payload by construction, to the extent the generating harness and adapter are qualified (Rule 12.2.5); the engine still recomputes the result (Rule 9.3.2).

When the VP's `derivation` is `authored` (explicitly linked to an existing procedure), the human must declare an evidence schema conforming to the constraint's measurement requirements. The evaluation engine validates that authored evidence conforms to the derived schema (metric and unit must match). Metric identity in this check is lexical (Rule 9.3.7).

**Derivation is mandatory where possible.** Manual evidence declarations are the escape hatch, not the default. When a harness exists for the obligation's domain, evidence must be produced through the derived path. The harness's qualification status is reported (Rule 12.2.5). The conformance checker emits a `derivable_but_linked` warning when manual evidence is provided for a VP that could have been derived.

#### 4.7.3 Verdict

|Field        |Type        |Description                                                                                                                                  |
|-------------|------------|---------------------------------------------------------------------------------------------------------------------------------------------|
|obligation   |ObligationID|Which obligation.                                                                                                                            |
|status       |enum        |`satisfied` (every demanded condition of O is `discharged`), `violated` (any condition is `failed`), `incomplete` (otherwise). See Rule 9.3.4.|
|coverage     |float       |\|discharged conditions of O\| / \|C(O)\| (Rule 9.3.5). Plan entries and methods do not multiply conditions.|
|blocking_gaps|list<Gap>   |Specific gaps preventing full satisfaction.                                                                                                  |

Verdict computation: For an obligation O, enumerate C(O) per Rule 9.3.1 (operating points × listed environments, with floor rules), before consulting any design. A condition (O, π, v) is discharged when, for every entry of O's verification plan that is bound to v (Section 4.8.2, `environments`), current evidence produced at π in v through a satisfying design passes on recomputation (Rule 9.3.2) and is admitted (Rule 10.2.5). Classify every condition per Rule 9.3.4.

#### 4.7.4 Gap

Gap types, their detection rules and their coverage buckets are defined in Section 9.1.

#### 4.7.5 RuntimeConstraint

|Field       |Type        |Description                                 |
|------------|------------|--------------------------------------------|
|id          |string      |Unique identifier.                          |
|derived_from|ObligationID|Source obligation.                          |
|check       |string      |The runtime assertion expression.           |
|on_violation|enum        |`safe_shutdown`, `degrade`, `log_and_alert`.|
|severity    |enum        |`safety`, `operational`, `monitoring`.      |

#### 4.7.6 TraceabilityChain

Not a stored artifact. The dependency DAG itself is the traceability. For any node in the graph, the chain is computed by traversal: Named source → Requirement (`record_id`) → Obligation → Design → VerificationProcedure → Evidence → Verdict. Every link is traversable in both directions.

#### 4.7.7 Interface (Computed View)

For any element pair (A, B), the pairwise interface view is the set of designs whose `components` include both A and B and that satisfy an obligation scoped `relational` whose `elements` are {A, B}, or, where that obligation declares no `elements`, that satisfy any relational obligation. Designs that satisfy emergent claims appear in a separate shared-medium view and are not interfaces.

An interface is not a declared instance type. It is a computed query over the design graph, the same way TraceabilityChain is a computed traversal of the dependency graph.

#### 4.7.8 VerificationRun

A derived, semi-authored type. Typically computed by the evaluation engine's run planner (Section 7.12) from the verification procedures, their footprints, and the environment's capabilities. Planner-derived runs and coverings are tool outputs (Rule 12.2.5). MAY be authored to record a run that was executed by hand. Its schema is normative for authored runs and for runs reported by the engine.

|Field                   |Type                                |Description                                                                                 |
|------------------------|------------------------------------|--------------------------------------------------------------------------------------------|
|id                      |string                              |Unique identifier.                                                                          |
|verification_environment|VerificationEnvironmentID           |Where the run executes.                                                                     |
|trajectory              |Trajectory                          |The maneuver timeline: the ordered path through S the run follows. See Section 4.7.8.1.     |
|windows                 |map<VerificationProcedureID, Window>|For each discharged VP, the sub-interval of the run during which the trajectory sits in the VP's required region.|
|instrumentation         |list<MeasurementChannel>            |The union of `payload_fields` across the evidence schemas of all discharged VPs. Guarantees the single capture contains every measurement every VP needs.|
|discharges              |list<VerificationProcedureID>       |The VPs this run produces evidence for.                                                     |
|operating_conditions    |list<OperatingConditionID>          |Computed: union of the operating points of the discharged procedures. Constrained by Rule 7.7.4.|
|artifact_reference      |string                              |The shared capture (telemetry log, recording, dataset) from which evidence is projected.    |
|derivation              |enum                                |`derived` (planner-generated) or `authored`.                                                |

**Feasibility.** A VerificationRun is well-formed only if its `windows` and `trajectory` satisfy the co-execution rules of Section 7.12 for its `discharges` set. An ill-formed run is a structural violation.

##### 4.7.8.1 Trajectory

A trajectory is an ordered list of segments. Each segment has a `region` (Section 5.3 grammar over declared dimensions), a `duration` bound, and for each constrained dimension an intent `hold`, `drive` or `observe` that MUST be consistent with effective controllability (Rule 4.5.1). Trajectories are distinct from Exchange interaction scenarios (Section 5.1) and from the Scenario gap type (Section 9.1).

### 4.8 Verification Methods and Verification Plans

#### 4.8.1 Verification Methods (TIAD)

OBSA recognizes four verification methods, consistent with the TIAD taxonomy established in ARP4754A, DO-178C, and INCOSE systems engineering practice:

|Method       |Description                                                   |Evidence Produced By                                                 |Examples                                                                    |
|-------------|--------------------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------|
|test         |Automated or manual execution that produces measurable results|Execution of a procedure against the system or component             |CI pact tests, integration tests, single-element tests of requirement-derived claims, dynamometer runs, flight tests|
|inspection   |Review of artifacts against criteria (checklist-based)        |Examination of physical or digital artifacts against defined criteria|Code review, wiring review, document review, dimensional inspection         |
|analysis     |Modeling, simulation, or expert assessment against a standard |Computational modeling, simulation, or domain expert evaluation      |CFD analysis, worst-case execution time analysis, stress analysis, FMEA     |
|demonstration|Observed execution showing capability works                   |Witnessed operation under defined conditions                         |Simulator evaluation, field demonstration, witnessed functional check            |

An analysis that is also a requirement source (for example an FMEA) MUST NOT be the sole evidence discharging a claim sourced from that same analysis.

The choice of verification method for a given parameter is authored by domain experts, not derived algorithmically. It reflects engineering judgment, applicable standards, and the nature of the constraint being verified.

**Informative note: test-internal pass criteria.**

Tests whose pass criterion is internal to the test or to the element's own design notes are developer practice and are not evidence by default. Any test, of any size, becomes evidence only through Rules 9.3.2–9.3.3.

**Informative note: model validation for analysis-derived evidence.**

When evidence is produced by the `analysis` method using a computational model, the validity of the model is itself an assurance concern. Model validation MAY be expressed as a separate obligation with constraints on model accuracy and evidence from comparison of model outputs against measurements from an environment or physical instrumentation in which the modeled quantity is directly observed. The general pattern is:

1. A primary obligation O requires evidence via `analysis` method.
2. A model validation obligation M is declared with O as parent.
3. M's constraint specifies acceptable model accuracy (e.g., prediction error ≤ 5%).
4. M's evidence comes from `test` method: comparison of model predictions against measurements in which the modeled quantity is directly observed.
5. O's analysis evidence is trustworthy because M is satisfied.

Model-validation obligations are parented per Rule 7.1.6.

#### 4.8.2 Verification Plan

A verification plan is a list of verification plan entries declared on both obligations and designs. Each entry maps a specific constraint parameter to a verification method.

**VerificationPlanEntry:**

|Field    |Type  |Required    |Description                                                                                                             |
|---------|------|------------|------------------------------------------------------------------------------------------------------------------------|
|parameter|string|yes         |A constraint metric or design shape field. Matched to the constraint metric lexically or through `maps_to` (Rule 9.3.7).|
|unit|string|no|Unit of the parameter, or "dimensionless" / "boolean".|
|method   |enum  |yes         |One of: `test`, `inspection`, `analysis`, `demonstration`.                                                              |
|maps_to  |string|no          |References a parameter in the obligation's VP. When present, establishes parameter mapping. When absent, string identity establishes coverage.|
|rationale|string|no          |Why this method was selected for this parameter.                                                                        |
|environments|list<VerificationEnvironmentID>|no|Obligation plans only. The listed environments in which this entry demands evidence; each MUST appear in the obligation's applicable_environments. Default: all of them. See Rule 7.7.7.|
|stimulus|enum|no|`active` (the measurement injects a command or disturbance) or `passive` (it observes without perturbing). When absent, treated as `active` under the conservative default of Rule 7.12.7.|
|drives|list<DimensionEffect>|no|The operating dimensions this entry's maneuver pushes, and in which direction. In an environment where a driven dimension is `settable` (Rule 4.5.1), the push is reset by the environment and induces no ordering.|
|sensor_channels|list<string>|no|The measurement channels this entry requires, with any incompatible configuration. Used for observability conflict detection (Rule 7.12.4).|

The `maps_to` field is an engineering assertion. It declares that the design-level parameter is the design-level realization of the referenced obligation-level parameter. The structural layer checks that the referenced parameter exists in the obligation's verification plan. It does not verify semantic equivalence; that relationship is domain knowledge, authored by the engineer. This is consistent with semantic-structural separation: structure checks the reference, meaning is asserted by the domain expert.

**Measurement footprint.** The `stimulus`, `drives` and `sensor_channels` fields are the entry's measurement footprint. They are authored engineering judgment. They are optional at Level 1, where their absence forces the conservative default of Rule 7.12.7, and expected at Level 2 for any entry whose procedures are intended to be batched into shared runs (Section 7.12).

**DimensionEffect:**

|Field    |Type  |Required|Description                       |
|---------|------|--------|----------------------------------|
|dimension|string|yes     |A declared OperatingDimension.    |
|direction|enum  |yes     |`increasing` or `decreasing`.     |

**Required region (derived).** A procedure's required region is the region of its operating point: the intersection of the regions of the conditions in that point (Rule 7.7.5). It is computed, not authored.

**Obligation verification plan:** Declares what parameters require evidence, by which methods, and in which of the obligation's listed environments (`environments`, Rule 7.7.7).

**Design verification plan:** Declares how the design's parameters will be verified. Must cover all parameters in the VPs of all obligations in the design's `satisfies` list.

**Mechanical matching rule:** For each entry in an obligation's VP, at least one entry in the satisfying design's VP must reference the same parameter (via `maps_to` or by string identity) with the same method. Checked as part of Level 1 conformance. This match selects evidence; it never determines the demanded condition set (Rule 9.3.1). A failed match surfaces as `no-design` or `no-evidence`, never as absence.

**Partial derivability note:** The minimum required (parameter, method) pairs on a design VP are derivable from the satisfied obligations' VPs. An implementation could generate this minimum set automatically. Additional entries beyond this minimum represent engineering judgment: the designer declaring verification work that goes beyond what the obligation structurally requires. The design VP is therefore partially transitional: its minimum content is derivable, but its full content may include authored additions.

### 4.9 TelemetryMapping (Stub)

A telemetry mapping declares that a runtime measurement source exists for an obligation's metric, enabling observability gap detection.

|Field     |Type        |Required|Description                                                                           |
|----------|------------|--------|--------------------------------------------------------------------------------------|
|id        |string      |yes     |Unique identifier.                                                                    |
|obligation|ObligationID|yes     |The obligation whose metric is observable.                                            |
|metric    |string      |yes     |The constraint metric being observed. Must match the obligation's constraint metric. Matched lexically (Rule 9.3.7).|
|source    |string      |yes     |The telemetry source: sensor ID, log stream, monitoring endpoint, data bus label, etc.|

**Cardinality:** One per observable obligation metric. Absence produces an observability gap (Section 9).

-----

## 5. Design Shapes

Designs are instantiated as one of the recognized shapes. Each shape models a distinct kind of relationship with distinct constraint types and distinct validation logic.

### 5.1 Exchange

Models structured transfer of data, energy, material, or force between components.

|Field    |Type          |Required|Description                                    |
|---------|--------------|--------|-----------------------------------------------|
|medium   |string        |yes     |The coupling medium. Domain-specific.          |
|flows    |list<Flow>    |yes     |Structured flow declarations. See Flow schema. |
|scenarios|list<Scenario>|no      |Behavioral sequences describing interaction patterns (request-response, cyclic, event-driven, failure annunciation). **Stub.** The Exchange scenario sub-schema (interaction patterns such as request-response or cyclic exchange) will be formalized in a future revision. It is unrelated to verification-run trajectories (Section 4.7.8.1) and to the Scenario gap type. For now, scenarios are unstructured annotations that inform harness design but are not structurally checked.|

**Flow:**

| Field    | Type        | Required | Description                                        |
|----------|-------------|----------|----------------------------------------------------|
| name     | string      | yes      | Identifier for this flow within the exchange.      |
| from     | ComponentID | yes      | The component that produces/sends this flow.       |
| to       | ComponentID | yes      | The component that consumes/receives this flow.    |
| protocol | string      | yes      | The protocol or physical medium carrying the flow. Domain-specific. See recommended values below. |
| encoding | string      | yes      | How the payload is encoded on the protocol. Domain-specific. |
| timing   | Timing      | no       | Timing characteristics. See below.                 |
| content  | string      | no       | Domain-specific payload description. Unstructured annotation for human readability and harness hints. |

The `from` and `to` fields reference Component instances (Section 4.6) declared in the design's `components` list. The structural check is: do `from` and `to` both reference components in this design's `components` field?

The `protocol` and `encoding` fields are strings, not enums. Their values are domain-specific. The structural check is: are they present and non-empty? Domain harnesses interpret them. This preserves structural parametricity: the evaluation function checks structure (fields present), not content (what protocol means).

**Recommended protocol values (informative):**

| Domain          | Protocol values                                           |
|-----------------|-----------------------------------------------------------|
| Space           | `SpaceWire`, `CCSDS:TM`, `CCSDS:TC`                      |
| Industrial      | `CAN`, `PROFINET`, `Modbus:TCP`, `Modbus:RTU`            |
| Aerospace       | `ARINC_429`, `ARINC_664`, `MIL-STD-1553`                 |
| Software        | `gRPC`, `REST:HTTP`, `MQTT`, `DDS:RTPS`, `shared_memory` |
| Mechanical      | `bolted_joint`, `spline_coupling`, `thermal_path`         |
| Electrical      | `power:DC`, `power:AC`, `analog_signal`, `discrete_IO`   |
| Fluid           | `hydraulic`, `pneumatic`, `coolant`                       |

**Recommended encoding values (informative):**

| Protocol family | Encoding values                                           |
|-----------------|-----------------------------------------------------------|
| ARINC 429       | `ARINC_429:BNR`, `ARINC_429:BCD`, `ARINC_429:discrete`    |
| Safety          | `black_channel:CRC32`, `redundant_pair`                   |
| Software API    | `protobuf`, `JSON`, `MessagePack`, `flatbuffers`         |
| Mechanical      | `torque:Nm`, `force:N`, `displacement:mm`                 |
| Electrical      | `voltage:VDC`, `current:A`, `PWM:duty_cycle`              |

**Timing:**

| Field   | Type   | Required | Description                                        |
|---------|--------|----------|----------------------------------------------------|
| period  | number | no       | Nominal period between transmissions.              |
| unit    | string | yes      | Time unit (us, ms, s).                             |
| jitter  | number | no       | Maximum timing deviation from nominal period.      |
| latency | number | no       | Maximum end-to-end latency from send to receive.   |
| timeout | number | no       | Maximum time before a missing update is an error.  |

**Domain examples:**

- FADEC → flight computer over ARINC 429: thrust parameter reports, failure annunciations. Flight computer → FADEC: thrust command words.
- Turbine disk → shaft over spline joint: torque transmission, axial load path. Shaft → disk: rotational constraint.
- Hydraulic pump → actuator over hydraulic line: pressure, flow rate. Actuator → pump: return flow, position feedback.
- Ground server → dispatch client over ACARS: weather updates, acknowledgments. Client → server: flight plan uplinks.
- Assembly station A → station B: part flow, quality handoff criteria.

NOTE: A flow over a dedicated point-to-point link is a relational claim between its two endpoints. A flow's timing, latency or bandwidth over a shared bus or network depends on the medium's other participants; it is an emergent claim naming them, typically satisfied by a Budget design. A property of one producer-consumer pair that does not depend on the medium (for example encoding or schema agreement) remains relational (Section 4.2).

**Typical constraint types:** timing, rate, ordering, conformance.

### 5.2 Compatibility

Models agreement on versions, parameters, tolerances, or specifications across a boundary.

|Field        |Type             |Required|Description                                                                                           |
|-------------|-----------------|--------|------------------------------------------------------------------------------------------------------|
|parameters   |list<CompatParam>|yes     |Named parameters that must agree. Each has: name, component_a_value, component_b_value, matching_rule.|
|matching_rule|enum             |yes     |One of: `exact`, `range`, `semver_compatible`, `tolerance_within`, `superset`.                        |

**Domain examples:**

- Flight computer software v4.2 ↔ FADEC firmware v2.8: failure mode encoding agreement
- Bolt pattern AN4-7A ↔ lug fitting AN4-7A: dimensional and material match
- Ground services API v3.2 ↔ dispatch client v3.1: semver backward compatibility
- Weld filler alloy ↔ base metal alloy: metallurgical compatibility per AWS D17.1
- Engine mount ↔ pylon attachment: bolt hole pattern and torque specification agreement

**Typical constraint types:** match, conformance.

### 5.3 Envelope

Models the boundary of validated capability, operating range, or characterization evidence across one or more dimensions.

**Scalar envelope fields (single dimension):**

|Field                    |Type      |Required   |Description                                                               |
|-------------------------|----------|-----------|--------------------------------------------------------------------------|
|dimension                |string    |yes        |What is being bounded: altitude, airspeed, temperature, thrust, Cpk, etc. |
|validated_range          |Range     |yes        |The range within which capability has been demonstrated.                  |
|characterization_method  |string    |no         |How the envelope was established (test, analysis, simulation, field data).|
|characterization_evidence|EvidenceID|no         |Reference to evidence establishing the envelope.                          |

**Multi-dimensional envelope fields:**

|Field                    |Type                             |Required   |Description                                                                                                                                             |
|-------------------------|---------------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|dimensions               |list<EnvelopeDimension>          |yes        |The parameters being jointly bounded. Each has: name, range ({min, max, unit}).                                                                         |
|interaction_constraints  |list<InteractionConstraint>      |no         |Constraints on the joint relationship between two or more dimensions. See below.                                                                        |
|representation           |enum                             |no         |One of: `hyperrectangle`, `hyperrectangle_with_interactions`, `convex_hull`, `parametric_boundary`. Defaults to `hyperrectangle` if no interactions declared, `hyperrectangle_with_interactions` if interactions are present.|
|characterization_method  |string                           |no         |How the envelope was established.                                                                                                                       |
|characterization_evidence|list<EvidenceID>                 |no         |References to evidence establishing the envelope.                                                                                                       |
|characterization_points  |integer                          |no         |Number of characterization points used to establish the envelope boundary.                                                                              |

An Envelope design uses EITHER the scalar fields (for a single dimension) OR the multi-dimensional fields (for two or more dimensions), not both.

**EnvelopeDimension:**

|Field |Type  |Required|Description                                  |
|------|------|--------|---------------------------------------------|
|name  |string|yes     |The parameter being bounded.                 |
|range |Range |yes     |The validated range: `{min, max, unit}`.     |

**InteractionConstraint:**

|Field     |Type        |Required|Description                                                                                                    |
|----------|------------|--------|---------------------------------------------------------------------------------------------------------------|
|parameters|list<string>|yes     |Names of the interacting dimensions. Must reference names declared in the `dimensions` list. At least two.     |
|expression|string      |yes     |A constraint expression over the named parameters. See expression grammar below.                               |
|decidable |boolean     |no      |Whether the expression is in the mechanically decidable fragment. Default: inferred from expression grammar.   |
|rationale |string      |no      |Why this interaction exists (physical coupling, empirical observation, safety analysis).                        |

**Expression grammar.**

Interaction constraint expressions are written over declared dimension names using arithmetic operators and comparison operators. The expression language has two fragments with different decidability properties:

**Mechanically decidable fragment (polynomial inequalities):**

Expressions where all terms are polynomial functions of the declared dimension names with constant (numeric literal) coefficients. This includes:

- Linear inequalities: `2 * surface_slope + 3 * payload_mass <= 5000`
- Bilinear terms: `surface_slope * payload_mass <= 5000`
- Quadratic terms: `temperature^2 + voltage^2 <= 2500`
- Higher-degree polynomials: `a * b * c <= threshold`
- Boolean combinations: `NOT (temperature > 45 AND voltage < 22)` where each atomic predicate is a polynomial inequality
- Comparison chains: `a <= b AND b <= c`

Containment checking for polynomial interaction constraints is decidable by quantifier elimination over the reals (Tarski, 1951). For the dimensionalities typical of engineered systems (d ≤ 20) and polynomial degrees typical of physical coupling models (degree ≤ 4), this is tractable. The evaluation engine's bounded evaluation property (Section 1.6) provides a practical backstop: if a specific instance exceeds the engine's time bound, it is treated as requiring analysis evidence; the affected conditions remain in the demanded set as `undecidable` (Rule 9.3.4).

**Non-polynomial fragment (analysis-deferred):**

Expressions containing transcendental functions (sin, cos, exp, log), implicit equations, or any function not expressible as a polynomial over the declared dimension names. Examples:

- `sin(temperature) * voltage > threshold`
- `exp(-altitude / 8500) * thrust > min_thrust`
- `lookup_table(temperature, humidity) > threshold`

For expressions in the non-polynomial fragment, the evaluation engine MUST NOT attempt mechanical containment checking. Instead, the design MUST include a verification plan entry with method `analysis` covering the containment assertion. The structural rule checks: (a) is the expression non-polynomial? (b) does the design's verification plan include an analysis entry for the containment parameter? If (a) and not (b), emit a structural conformance violation. If (a) and (b), the containment assertion is discharged by analysis evidence rather than mechanical checking. Until analysis evidence for the containment assertion is admitted, the affected conditions remain in the demanded set as `undecidable` (Rule 9.3.4).

The `decidable` field allows the author to explicitly mark an expression as non-polynomial when the grammar classification is ambiguous. When `decidable` is absent, the engine infers decidability from the expression syntax. When `decidable: false` is present, the engine skips mechanical checking and requires analysis evidence regardless of expression syntax. When `decidable: true` is present, the engine attempts mechanical checking.

**Representation semantics:**

- `hyperrectangle`: The validated region is the Cartesian product of the per-dimension ranges. Each dimension is independent.
- `hyperrectangle_with_interactions`: The validated region is the Cartesian product of per-dimension ranges intersected with the interaction constraints.
- `convex_hull`: The validated region is the convex hull of characterization points.
- `parametric_boundary`: The validated region is defined by a level set of a function.

**Domain examples:**

- Engine thrust curve: sea level to 45,000 ft, -40°C to ISA+35°C, validated by dynamometer and flight test (multi-dimensional, temperature and altitude interact)
- Flight envelope: VMCA to VMO/MMO, validated across CG range and weight range
- Manufacturing process capability: Cpk ≥ 1.67 for turbine blade root form grinding at ±0.005mm (scalar)
- Ground system SLA: p99 latency ≤ 200ms validated at up to 50,000 concurrent connections (scalar)
- Battery discharge envelope: validated from -40°C to 55°C, 10%-100% SoC, with interaction constraint that discharge rate is coupled to temperature
- Multi-axis operating envelope: validated across temperature, supply voltage, vibration, and altitude with interactions between temperature and altitude affecting component derating

NOTE: An Envelope design MAY be referenced by different obligations in different mode states (see Section 5.4 and Rule 7.10).

**Typical constraint types:** containment, containment-nd, threshold, stability.

### 5.4 Mode

Models valid states, transitions, guards, and invariants for a stateful element.

|Field              |Type                              |Required|Description                                                                                                  |
|-------------------|----------------------------------|--------|-------------------------------------------------------------------------------------------------------------|
|states             |list<State>                       |yes     |Valid states. Each has: name, invariants (obligations that apply in this state), entry_actions, exit_actions, `terminal` (boolean, default false; absorbing within a run, see Rule 7.12.5).|
|transitions        |list<Transition>                  |yes     |Valid transitions. Each has: source, target, trigger, guard (condition that must hold), actions.             |
|initial_state      |string                            |yes     |The starting state.                                                                                          |
|mode_obligation_map|map<StateName, list<ObligationID>>|no      |Which obligations apply in which states. Obligations not in the map apply in ALL states. See Rule 7.10.     |

A state with `terminal: true` is absorbing: once entered, it cannot be exited within a single run without a reset. A verification whose maneuver drives the system into a terminal state partitions the run (Rule 7.12.5): everything requiring the pre-terminal state must precede it, and the run ends (or a new run is required) after it. Examples of terminal states: a fall-containment state, a latched safe-torque-off, a destructive-test end state.

**Domain examples:**

- Flight phases: preflight → taxi → takeoff_roll → initial_climb → cruise → approach → landing → rollout
- Engine controller modes: off → starting → idle → climb_thrust → cruise_thrust → reverse_thrust → shutdown → fault
- Manufacturing line states: setup → running → changeover → maintenance → idle
- Ground services lifecycle: starting → healthy → degraded → draining → terminated
- Landing gear states: up_and_locked → transit_down → down_and_locked → weight_on_wheels

**Typical constraint types:** ordering, boolean, timing.

### 5.5 Budget

Models a finite resource with supply, demand, allocation, and capacity constraints.

|Field        |Type            |Required|Description                                                                                                        |
|-------------|----------------|--------|-------------------------------------------------------------------------------------------------------------------|
|resource     |string          |yes     |What is budgeted: thrust, weight, power, compute, fuel_flow, bandwidth, cost, cycle_time, thermal_dissipation, etc. Shared-medium resources (bus bandwidth, bus timing) and additive totals over three or more elements are emergent claims satisfied by Budget designs (Rule 7.2.4).|
|unit         |string          |yes     |Unit of measurement.                                                                                               |
|supply       |number          |yes     |Total available quantity.                                                                                          |
|allocations  |list<Allocation>|yes     |Per-component demands. Each has: component, demand, priority (optional).                                           |
|margin_policy|MarginPolicy    |no      |Required reserve: {minimum_margin_percent, rationale}.                                                             |

**Domain examples:**

- Weight budget: 85,000 kg MTOW allocated across airframe, engines, systems, payload, fuel, margin
- Electrical power budget: 150 kVA total from generators, allocated across avionics, flight controls, galley, etc.
- Manufacturing cycle time: 120s per blade, allocated across grinding, inspection, deburring

**Typical constraint types:** balance, threshold.

### 5.6 Open Shape Membership

The five shapes above represent relationships encountered in the systems examined to date. This set is explicitly open. A new shape MAY be added when ALL of the following conditions are met:

**Rule 5.6.1.** The relationship requires validation logic that cannot be expressed using any existing shape's constraint types and structure.

**Rule 5.6.2.** The new shape has at least one constraint type that is structurally distinct from all constraint types used by existing shapes, OR applies existing constraint types in a structurally different validation pattern.

**Rule 5.6.3.** The new shape is documented with: name, relationship modeled, shape-specific fields, applicable constraint types, at least three domain examples spanning at least two engineering disciplines, and a justification that existing shapes cannot model the relationship.

Adding a shape does not invalidate existing declarations.

-----

## 6. Constraint Types

Each constraint type defines an assertion pattern, a refinement rule, and an acceptance function. The refinement rule specifies what "at least as strict" means for that type, enabling mechanical checking of whether a design satisfies an obligation. The acceptance function specifies how to evaluate evidence against the constraint, enabling mechanical verdict computation.

### 6.1 Recognized Constraint Types

|ID               |Type                      |Assertion Pattern                         |Refinement Rule                                                           |Acceptance Function                                   |
|-----------------|--------------------------|------------------------------------------|--------------------------------------------------------------------------|------------------------------------------------------|
|CT-TIMING        |Timing                    |duration ≤ threshold                      |design_threshold ≤ obligation_threshold                                   |measured_duration ≤ design_threshold                  |
|CT-RATE          |Rate                      |frequency ≥ target                        |design_target ≥ obligation_target                                         |measured_frequency ≥ design_target                    |
|CT-THRESHOLD     |Threshold                 |value ∈ [min, max]                        |design_range ⊆ obligation_range                                           |measured_value ∈ design_range                         |
|CT-MATCH         |Match                     |value_a = value_b                         |Exact equality. No relaxation.                                            |observed_value = specified_value                      |
|CT-ORDERING      |Ordering                  |events occur in sequence S                |design_sequence preserves obligation_sequence ordering                    |observed_sequence preserves design_sequence ordering  |
|CT-CONTAINMENT   |Containment               |observed ⊆ validated                      |design_validated_range ⊇ obligation_required_range                        |observed_value ∈ design_validated_range               |
|CT-CONTAINMENT-ND|N-Dimensional Containment |required_region ⊆ validated_region in ℝⁿ  |design_validated_region ⊇ obligation_required_region                      |evidence_point ∈ design_validated_region              |
|CT-BOOLEAN       |Boolean                   |condition = true                          |No relaxation. Must be true.                                              |observed_condition = true                             |
|CT-BALANCE       |Balance                   |Σ(demands) ≤ supply                       |design_supply ≥ obligation_supply AND all demands covered                 |measured_supply ≥ Σ(measured_demands)                 |
|CT-STABILITY     |Stability                 |Δ(value, interval) ≤ threshold            |design_threshold ≤ obligation_threshold                                   |measured_Δ ≤ design_threshold                         |
|CT-CONFORMANCE   |Conformance               |instance matches spec                     |design_spec ⊇ obligation_spec (at least as strict in all dimensions)      |observed_instance conforms to design_spec             |
|CT-STATISTICAL   |Statistical               |statistic(metric, sample) ∈ [min, max]    |design_range ⊆ obligation_range on the statistic                          |computed_statistic ∈ design_range                     |

The acceptance function evaluates evidence at the *design* constraint level. Because the design constraint refines the obligation constraint (Rule 7.3), evidence satisfying the design's acceptance function necessarily satisfies the obligation's constraint by transitivity. (The engine applies this function itself; Rule 9.3.2.)

### 6.2 Constraint Structure and Measurement Payload

Every constraint instance has:

|Field   |Type                    |Required|Description                             |
|--------|------------------------|--------|----------------------------------------|
|type    |ConstraintTypeID        |yes     |Which constraint type.                  |
|metric  |string                  |yes     |What is being measured. Metric identity is lexical in this revision (Rule 9.3.7).|
|operator|string                  |yes     |The comparison operator (type-specific).|
|value   |number, string, or range|yes     |The threshold, target, or specification.|
|unit    |string                  |yes     |Unit of measurement.                    |

Every evidence measurement payload has:

|Field |Type                              |Required|Description                                                                                                        |
|------|----------------------------------|--------|-------------------------------------------------------------------------------------------------------------------|
|metric|string                            |yes     |What was measured. Must match the constraint metric.                                                               |
|value |number, string, range, or sequence|yes     |The measured, observed, or assessed result.                                                                        |
|unit  |string                            |yes     |Unit of measurement. Must match the constraint unit.                                                               |

The acceptance function for a constraint type takes a measurement payload and returns `pass`, `fail`, or `inconclusive`.

**Per-constraint-type evidence measurement payloads:**

The base measurement payload (`metric`, `value`, `unit`) is extended with type-specific fields for the following constraint types:

|Constraint Type |Additional Measurement Fields                                                                                     |
|----------------|------------------------------------------------------------------------------------------------------------------|
|CT-BALANCE      |`supply_measured: number`, `demands_measured: [{component: ComponentID, value: number}]`. `value` is `bool` (budget balanced).|
|CT-TIMING       |`value: number`. The measured timing value in the constraint's unit.                                               |
|CT-THRESHOLD    |`value: number`. The measured value in the constraint's unit.                                                      |
|CT-CONTAINMENT  |`value: {dimensions: [{name: string, value: number}]}`. The measured point in the design's validated region.      |
|CT-STATISTICAL  |`value: number`, `sample_size: integer`, `observation_window: string`. The computed statistic over the sample.    |

For constraint types not listed above (CT-BOOLEAN, CT-MATCH, CT-ORDERING, CT-STABILITY, CT-CONFORMANCE, CT-RATE), the base measurement payload is sufficient. The `value` field type matches the constraint's `value` type.

**Statistical constraint additional fields:** CT-STATISTICAL constraints extend the base constraint structure with:

|Field              |Type   |Required|Description                                                                                               |
|-------------------|-------|--------|----------------------------------------------------------------------------------------------------------|
|statistic          |enum   |yes     |The summary statistic: `p50`, `p90`, `p95`, `p99`, `p999`, `mean`, `cpk`, `ppk`, `sigma_level`, `std_dev`.|
|sample_size_minimum|integer|no      |Minimum number of observations required for a valid evaluation.                                           |
|observation_window |string |no      |Time window over which observations are collected.                                                        |
|confidence_level   |float  |no      |Required confidence level for the statistic.                                                              |

**Rule 6.2.1 (statistical independence under co-execution).** When a CT-STATISTICAL VP is discharged across co-executed runs, the effective number of independent samples is bounded by the number of **distinct runs**, not by the number of maneuvers or windows within a run. Samples harvested from one run's trajectory are correlated. The evaluation engine MUST compute `sample_size` for a statistical verdict from distinct-run count when co-execution is used, and MUST NOT sum within-run samples as if independent. A statistical obligation whose `sample_size_minimum` exceeds the available distinct-run count has an evidence gap, not a satisfied verdict. The engine-computed effective sample size takes precedence over a reported `sample_size` (as Rule 7.11.2 does for computed fields); both are reported. An insufficient effective sample places the condition in `no-evidence` (Rule 9.3.4).

**N-dimensional containment constraint additional fields:** CT-CONTAINMENT-ND constraints extend the base constraint structure with:

|Field                  |Type                       |Required|Description                                                                                                                 |
|-----------------------|---------------------------|--------|----------------------------------------------------------------------------------------------------------------------------|
|dimensions             |list<DimensionSpec>        |yes     |The dimensions of the required region. Each has: name, range ({min, max, unit}).                                            |
|interaction_constraints|list<InteractionConstraint>|no      |Constraints on joint relationships between dimensions. See Section 5.3.                                                     |
|representation         |enum                       |no      |One of: `hyperrectangle`, `hyperrectangle_with_interactions`, `convex_hull`, `parametric_boundary`. Default: inferred.      |

Interaction constraint expressions are subject to the expression grammar defined in Section 5.3. Expressions in the mechanically decidable fragment (polynomial inequalities) are checked mechanically during refinement. Expressions in the non-polynomial fragment require analysis evidence. See Section 5.3 for the full grammar specification and the decidability boundary.

**N-dimensional containment refinement rule:**

The design's validated region MUST contain the obligation's required region. Containment checking depends on the representations used and on the expression grammar of any interaction constraints.

**When all interaction constraints are in the decidable fragment:**

- **Both hyperrectangles (with or without interactions):** Containment holds if every vertex of the obligation's required region (intersected with its interaction constraints) is contained within the design's validated region (intersected with its interaction constraints). For polynomial interaction constraints, this is decidable by quantifier elimination. For linear interaction constraints, this reduces to a finite set of linear programs.

- **Obligation hyperrectangle, design convex hull:** Containment holds if every vertex of the obligation's required region is inside the design's convex hull. Point-in-hull is a linear program with N constraints (N = number of hull-defining points) and d variables.

- **Obligation hyperrectangle, design parametric boundary:** Containment holds if the minimum of the boundary function over the obligation's required region meets or exceeds the threshold. This is a bounded optimization problem. If the boundary function is concave over the required region, the minimum is at a vertex.

For all representations with decidable-fragment interaction constraints, the containment check terminates with a definite verdict for the dimensionalities and polynomial degrees typical of engineered systems. The evaluation engine's bounded evaluation property (Section 1.6) provides a backstop for pathological instances.

**When any interaction constraint is in the non-polynomial fragment:**

The evaluation engine MUST NOT attempt mechanical containment checking. The containment assertion is discharged by analysis evidence declared in the design's verification plan (see Section 5.3 expression grammar). The structural rule checks that such evidence is declared and, at Level 3 conformance, that it is present and its result, computed under Rule 9.3.2, is `pass`.

**N-dimensional containment acceptance function:**

An evidence point (a measurement in ℝⁿ at specific operating condition values) passes if the point falls within the design's validated region.

**Region intersection.** Two regions intersect if their per-dimension intervals overlap on every shared dimension and the conjunction of their interaction constraints is satisfiable. Where interaction constraints fall outside the mechanically decidable fragment (Section 5.3), the intersection is *undecidable*.

**Relationship between CT-CONTAINMENT and CT-CONTAINMENT-ND:**

CT-CONTAINMENT is the single-dimensional case. CT-CONTAINMENT-ND is the general multi-dimensional case. Implementations MAY treat CT-CONTAINMENT as syntactic shorthand for the single-dimensional CT-CONTAINMENT-ND case.

### 6.3 Open Constraint Type Membership

The constraint types above represent assertion patterns encountered to date. This set is explicitly open. A new constraint type MAY be added when:

**Rule 6.3.1.** The assertion pattern cannot be expressed as an instance of any existing constraint type.

**Rule 6.3.2.** The new type has a defined refinement rule specifying what "at least as strict" means.

**Rule 6.3.3.** The new type has a defined acceptance function specifying how evidence is evaluated.

**Rule 6.3.4.** The new type is documented with: ID, assertion pattern, refinement rule, acceptance function, at least two domain examples.

-----

## 7. Structural Rules

A declaration set is *structurally conformant* if and only if all of the following rules are satisfied.

### 7.1 Traceability

**Rule 7.1.1.** Every Obligation with status `active` MUST reference exactly one derivation parent in its `parent` field, which is either a Requirement ID or a parent Obligation ID. This is the obligation's authoring lineage: the requirement or parent obligation from which it was decomposed.

An obligation MAY additionally appear in the `children` list of other obligations, including obligations in other declaration sets during composition. These are compositional contributions, not derivation parents. An obligation's `parent` field is unique. Its compositional contributions are unbounded. The composed dependency graph is a DAG (directed acyclic graph) where the `parent` field defines derivation edges and the `children` lists define compositional edges. Both edge types are acyclic (Rule 7.1.4).

**Rule 7.1.2.** Every Obligation with status `active` MUST be reachable from at least one Requirement by traversing parent references upward. No orphan obligations. The Requirement reached MUST satisfy Rule 7.1.5.

**Rule 7.1.3.** Every Design MUST reference at least one Obligation with status `active` in its `satisfies` field.

**Rule 7.1.4.** The parent reference graph of obligations MUST be acyclic (a directed acyclic graph).

**Rule 7.1.5 (requirement source).** Every Requirement MUST declare a `source_type` (Section 4.1) and a `source_reference` naming a source artifact that is not a verification procedure, evidence entry, test, or verification plan of the declaration set. A test result is never a source. A Requirement without a `record_id` is reported as an Unrecorded Requirement gap (Section 9.1). Violation is a structural conformance failure.

**Rule 7.1.6 (origin routing).** An obligation whose `source_type` is `test_failure`, `field_discovery` or `manufacturing_nonconformance` MUST have an ancestor Requirement whose source kind is `capa` or `fmea`. A model-validation obligation (Section 4.8.1) MUST parent to the obligation whose evidence depends on the model.

**Rule 7.1.7 (assurance priority, informative SHOULD).** Assurance work SHOULD be prioritized hazard analysis, then FMEA, then CAPA. This orders work; it does not restrict which source kinds are admitted.

### 7.2 Scope Consistency

**Rule 7.2.1.** An obligation with scope `intrinsic` concerns one element (Section 4.2). A satisfying design MAY involve further, contextual components.

**Rule 7.2.2.** An obligation with scope `relational` concerns the coupling between two elements and MUST be satisfied by designs whose `components` number two or more.

**Rule 7.2.3.** An obligation with scope `emergent` concerns three or more elements and a property not attributable to any one of them or to any single pairwise interface. It MUST be satisfied by a design whose components collectively span the boundary relevant to the property. Non-attribution is an authored judgment; a `rationale` SHOULD state it.

**Rule 7.2.4 (elements).** For every obligation: (a) the number of its `elements` MUST match the scope class (`intrinsic` 1, `relational` 2, `emergent` 3 or more), and (b) each satisfying design's `components` MUST include every element. Attribution, including whether a claim depends on a shared medium (Section 4.2), is not checked structurally (Section 3.1). `elements` is required (Section 4.2).

### 7.3 Refinement

**Rule 7.3.1.** For every obligation O referenced in a design D's `satisfies` list, D MUST declare at least one constraint of the same type as O's constraint.

**Rule 7.3.2.** Each such design constraint MUST be at least as strict as the corresponding obligation constraint, as defined by the refinement rule for that constraint type (Section 6.1).

**Rule 7.3.3.** If a design constraint is not at least as strict as its obligation constraint, the declaration set is non-conformant and the specific violation MUST be reported.

**Rule 7.3.4.** Refinement establishes transitivity of evidence: evidence satisfying a design's constraint necessarily satisfies the obligation's constraint, because the design's constraint is at least as strict.

Refinement matches by constraint type, not by metric name. A design constraint of type CT-THRESHOLD on metric `airspeed_tracking_error` refines an obligation constraint of type CT-THRESHOLD on metric `airspeed_hold_accuracy` if the design's range satisfies the refinement rule for CT-THRESHOLD. The relationship between the design metric and the obligation metric (that airspeed tracking error is the design-level realization of airspeed hold accuracy) is carried by the `maps_to` field on the design's verification plan entry (Section 4.8.2). Rule 7.3 checks type-level refinement on values. Rule 7.9.3 checks parameter coverage via `maps_to` references. Metric identity across the `maps_to` join is lexical in this revision (Rule 9.3.7). Two rules, two concerns, no overlap.

### 7.4 Obligation Decomposition Consistency

**Rule 7.4.1.** If an obligation O has children C₁, C₂, …, Cₙ, the children's constraints MUST be consistent with O's constraint as follows:

- For **timing** obligations: Σ(child timing values) ≤ parent timing value (timing budgets close).
- For **balance** obligations: Σ(child supply values) ≤ parent supply value AND child allocations partition the parent resource.
- For **match** obligations: all children must reference the same matched value.
- For **threshold** obligations: each child's range must be contained within the parent's range.
- For **containment** obligations (CT-CONTAINMENT and CT-CONTAINMENT-ND): child validated regions, projected onto the parent's dimensions, must collectively cover the parent's required region.
- For other types: the composition rule is type-specific. If no explicit composition rule is defined for a constraint type, the parent obligation MUST include in its verification plan an `analysis` or `inspection` method entry covering the composition assertion.

Children join to the parent on the same metric (Rule 9.3.7) and MUST use consistent units.

**Rule 7.4.2.** If an obligation O has children, the union of the children's named elements, pairwise couplings and emergent element sets MUST cover the elements and couplings relevant to O. This is the *completeness* property. Incomplete decomposition is a structural warning (not a conformance violation) that MUST be reported.

### 7.5 Cross-Obligation Consistency

**Rule 7.5.1.** If two obligations O₁ and O₂ with status `active` apply to the same component (they name a common element) and have constraints of the same type on the same metric (Rule 9.3.7), the constraints MUST NOT be contradictory.

**Rule 7.5.2.** Contradictory obligations MUST be reported as structural errors. Resolution requires either modifying one obligation or introducing a mode design that separates the contexts in which each applies.

### 7.6 Safety Obligations

**Rule 7.6.1.** An obligation derived (directly or transitively) from a requirement whose `source_type` is `hazard_analysis` (or its deprecated alias `safety_analysis`) is a *safety obligation*.

**Rule 7.6.2.** Safety obligations MUST include every declared operating condition in their `applicable_conditions` list. This is the explicit form of "safety obligations apply under all conditions." A safety obligation's operating points are formed from all declared conditions (Rule 7.7.5). A declared subset (Rule 7.7.6) MAY restrict them; for a safety obligation the subset's rationale MUST cite the analysis that justifies each omitted point, and the conformance report lists every safety obligation that uses a subset.

**Rule 7.6.3.** [Withdrawn in 0.2.2 with the fidelity ordering. See Rule 7.6.5.]

**Rule 7.6.4.** Safety obligations MUST NOT have status `withdrawn` without reference to a revised hazard-analysis record (cited by its `record_id` where the Requirement declares one) that removes the underlying hazard or modifies its severity classification. This reference MUST be recorded in a rationale field.

**Rule 7.6.5 (SHOULD).** For each metric a safety obligation constrains, at least one environment whose `realization` is `physical` or `mixed` (Section 4.5) SHOULD be bound (Section 4.8.2, Rule 7.7.7) to an entry of the obligation's verification plan whose `parameter` is that metric (Rule 9.3.7) and, where that environment declares `observable_metrics`, the metric is among them, and the obligation's rationale SHOULD state why each listed environment is credible for that quantity. The engine reports a warning for each constrained metric with no such binding, including when an edit to `applicable_environments` or to an entry's `environments` removes the last one. The appropriate environments depend on the integrity level and applicable standard (ARP4754A, DO-178C, DO-254).

### 7.7 Operating Conditions and Verification Environments

**Rule 7.7.1.** Every obligation with status `active` MUST declare `applicable_conditions` containing at least one OperatingConditionID. Every referenced ID MUST correspond to a declared OperatingCondition instance.

**Rule 7.7.2.** Every obligation with status `active` MUST declare `applicable_environments` containing at least one VerificationEnvironmentID. Every referenced ID MUST correspond to a declared VerificationEnvironment instance. Every listed environment is demanded. Environment capability MAY validate or propose additions to this list (Section 10.3, Rule 10.3.1); it never removes environments from it.

**Rule 7.7.3.** The verification matrix is the demanded condition set C of Rule 9.3.1: (obligation, operating point, environment) triples, computed from obligations alone. Verification plan entries and methods do not multiply C.

**Rule 7.7.4.** The operating-condition set of a verification run MUST be pairwise **compatible** (Rule 7.12.1) at every instant of the run. Two conditions required simultaneously (in overlapping windows) whose regions have an empty or undecidable intersection (Section 6.2) are a structural violation of the run. Such conditions MAY be visited in non-overlapping windows of the same trajectory, subject to Section 7.12.

**Rule 7.7.5 (operating points).** Let D(O) be the set of dimensions constrained by O.`applicable_conditions`. An operating point π is a set of applicable conditions such that every dimension in D(O) is constrained by exactly one condition in π. Π(O) is the set of all such points, unless restricted by Rule 7.7.6. An applicable condition that belongs to no operating point is reported as a warning. For scalar-form conditions (Rule 4.4.2) this is exactly one condition per dimension.

**Rule 7.7.6 (declared subset).** When `operating_point_selection.mode = subset`, Π(O) is the declared `points`, each of which MUST be an element of the full Π(O), and `rationale` is required. The subset is authored engineering judgment.

**Rule 7.7.7 (plan-entry environments).** Every environment named in an obligation plan entry's `environments` MUST appear in the obligation's `applicable_environments`, and every listed environment MUST be bound by at least one entry of the obligation's verification plan (Section 4.8.2).

### 7.8 Component References

**Rule 7.8.1.** Every Component ID referenced by any declaration MUST correspond to a Component instance in the declaration set, and every dimension name MUST resolve (Rule 4.4.1).

### 7.9 Verification Plan Coverage

**Rule 7.9.1.** Every obligation MUST declare a verification plan with at least one entry.

**Rule 7.9.2.** Every design MUST declare a verification plan with at least one entry.

**Rule 7.9.3.** For each obligation O in a design D's `satisfies` list, every parameter in O's verification plan MUST be covered by at least one entry in D's verification plan with the same verification method. Coverage is established by the design VP entry's `maps_to` field if present, or by string identity between design and obligation parameter names if `maps_to` is absent (lexical identity, Rule 9.3.7). If neither condition holds, coverage is missing. This is the *verification plan coverage* property. Missing coverage is a structural conformance violation.

**Rule 7.9.4.** Within a single verification plan (on either an obligation or a design), the pair (parameter, method) MUST be unique. Duplicate entries with the same parameter and method are a structural conformance violation. The evaluation engine MUST report duplicates and MUST NOT silently merge them.

### 7.10 State-Dependent Obligation Activation

**Rule 7.10.1.** If a Mode design declares a `mode_obligation_map`, then for each state S in the mode, the obligations mapped to S MUST be structurally conformant as an independent subset: traceability (Rule 7.1, including 7.1.5), scope consistency (Rule 7.2), refinement (Rule 7.3), and cross-obligation consistency (Rule 7.5) MUST hold within each state's obligation set.

**Rule 7.10.2.** Obligations not appearing in any `mode_obligation_map` entry apply in ALL states of that mode. These are *state-invariant* obligations.

**Rule 7.10.3.** Obligations appearing in a `mode_obligation_map` entry apply ONLY in the listed states. The obligation's `applicable_conditions` and `applicable_environments` lists define the operating points (Rule 7.7.5) and environments in which evidence is demanded, scoped to the states in which the obligation is active.

**Rule 7.10.4.** Cross-obligation consistency (Rule 7.5) is checked per-state. Two obligations that would be contradictory if both active simultaneously are permitted if they are mapped to non-overlapping states.

**Rule 7.10.5.** When a Mode design's `mode_obligation_map` activates different Envelope obligations in different states, the envelope applicable in each state defines the validated operating boundary for that state.

**Rule 7.10.6.** Gap analysis and verdict computation for state-specific obligations operate within the scope of each state. The obligation's declared `applicable_conditions` and `applicable_environments` define the operating points (Rule 7.7.5) and environments in which evidence is demanded, evaluated only in states where the obligation is active. The verification matrix for the system is the union of the per-state verification matrices.

NOTE: State-dependent obligation activation is the general mechanism for modeling systems where constraints change with operating phase.

### 7.11 Computed Fields

**Rule 7.11.1.** The following fields are computed, not authored. If present in a declaration, a conforming evaluation engine MUST verify them against the computed value and report discrepancies. If absent, the engine computes them.

- `children` on obligations: computed as the set of obligation IDs whose `parent` field references this obligation.
- `satisfied_by` on obligations: computed as the set of design IDs whose `satisfies` field references this obligation.
- `decomposed_into` on requirements: computed as the set of obligation IDs whose `parent` field references this requirement.

**Rule 7.11.2.** A discrepancy between an authored computed field and the engine-computed value is a structural warning, not a conformance violation. The engine-computed value takes precedence. The engine is a tool whose qualification status is reported (Rule 12.2.5).

### 7.12 Co-execution and Interference

This section defines the relations that govern whether verification procedures may share a run, and in what order. Every relation is **computed** from region geometry (Section 4.4, Section 5.3), effective controllability (Rule 4.5.1) and dimension dynamics (Section 4.4a), Mode terminal states (Section 5.4), and VP footprints (Section 4.8.2). None is authored pairwise. This preserves structural parametricity (Section 3.1): the relations operate on regions and type annotations, never on the domain meaning of a dimension.

Rules 7.12.1–7.12.8 are Level 1 checks on authored runs. For engine-derived runs and coverings they are engine obligations reported at Level 2. Throughout this section, *the run's environment* is the VerificationRun's `verification_environment` (Section 4.7.8).

Let A and B be verification procedures with required regions R_A, R_B (derived per Section 4.8.2) and footprints as declared.

**Rule 7.12.1 (compatible).** A and B are *compatible* if and only if R_A ∩ R_B ≠ ∅ (Section 6.2, region intersection). An undecidable intersection is treated as not compatible. Compatible procedures MAY occupy the same window of a run.

**Rule 7.12.2 (exclusive).** A and B are *instantaneously exclusive* if and only if they are not compatible (Rule 7.12.1). Exclusive procedures MUST NOT share a window. If, on every dimension of disjointness, the effective controllability in the run's environment (Rule 4.5.1) is `settable`, they MAY occupy different windows of one run in either order, subject to reachability (Rule 7.12.8(b)).

**Rule 7.12.3 (precedes: driven ordering).** Let d be a dimension whose effective controllability in the run's environment is `driven` (Rule 4.5.1). If A and B are exclusive on d, or B.`drives` moves d out of R_A, then a forced order holds: the procedure whose required region lies on the source side of d's direction (its `dynamics.direction`, or, absent dynamics, the conservative default of Rule 7.12.7) MUST precede the other within any shared run. A run's `windows` assignment MUST respect all such precedences.

**Rule 7.12.4 (stimulus and observability conflict).** If A.`stimulus = active` and B.`stimulus = passive`, and they share any named element (`elements`, or the satisfying designs' `components`) within overlapping windows, they conflict and MUST be assigned non-overlapping windows. If A and B require the same channel from their `sensor_channels` in incompatible configurations within overlapping windows, they cannot co-execute in those windows at all.

**Rule 7.12.5 (partitions: terminal transition).** If A's maneuver drives the system into a `terminal` Mode state (Section 5.4) or moves a dimension whose effective controllability in the run's environment is `driven` (Rule 4.5.1) and whose dynamics are `reversible_within_run = false` (or, absent dynamics, are treated so under Rule 7.12.7), such that R_B is unreachable (Rule 7.12.8(b)) afterwards, then A *partitions* the run relative to B: A MUST follow every procedure requiring the pre-transition state, and the run MUST end after A. B, if it requires the post-transition state to be exited, belongs to a different run.

**Rule 7.12.6 (regime dimensions and Mode).** Exclusivity on a regime dimension realized by a Mode design is the state-dependent activation of Rule 7.10. Where both a regime OperatingDimension and a Mode `mode_obligation_map` describe the same discrete axis, the Mode design is authoritative and the regime dimension is a projection of it. The co-execution relations above apply uniformly to discrete regime dimensions and continuous dimensions.

**Rule 7.12.7 (conservative default).** Absence of a footprint sufficient to prove non-interference is treated as potential interference. Specifically: a VP with no declared `stimulus` is treated as `active`; two VPs sharing components with no evidence of independence are treated as conflicting; a `driven` dimension with no declared `dynamics` is treated as irreversible and monotone. Under this default the procedures are serialized into separate windows or runs. Independence is *earned* by declaring the footprint, or by a cross-check (an unbatched confirmation run, or a passing run of the same plan entry at the same operating point in another listed environment in which the two procedures cannot interact). This direction is required because a missed interference yields a false pass, which is the unsafe failure mode.

**Rule 7.12.8 (feasibility: reachability, not only containment).** A set V of verification procedures is *co-executable in one run* in environment E if and only if there exists a trajectory through S and a window assignment such that:

(a) **Containment.** Every VP's required region is occupied during its window, and E can hold that region: each constrained dimension's effective controllability in E (Rule 4.5.1) is `settable`, `driven`, or `observed` such that the region is maintainable. This is a capability predicate; it does not establish that the environment's verdicts are trustworthy (Rule 10.2.6).

(b) **Reachability.** E can *steer* the trajectory through the ordered window sequence within the effective controllability of its dimensions (Rule 4.5.1). For `settable`/`instantaneous` dimensions the transition between windows is free (state may be set directly). For `settable`/`rate_limited` and `driven` dimensions the transition MUST be a path the system's own dynamics admit: the trajectory between windows is itself a segment through S, subject to Rules 7.12.1–7.12.7, along which `driven` dimensions (Rule 4.5.1) evolve per their `dynamics`. A window ordering that requires an unreachable transition (for example, instantaneously jumping a `rate_limited` dimension, or traversing a region excluded by an interaction constraint) is infeasible.

(c) **Rules.** Rules 7.12.1–7.12.7 hold across both windows and the transitions between them.

(d) **Observability.** Where E declares `observable_metrics`, every VP's evidence schema is producible from them (Rule 9.3.7).

The evolution of `driven` dimensions, both within windows and along between-window transitions, MUST be consistent with the declared `dynamics` (or their model-validation obligation, Section 4.4a note). Containment alone (an environment that can hold each region) does not establish feasibility; the environment must be able to *drive the run*, which is the reachability condition (b). An environment in which every constrained dimension is `settable`/`instantaneous` satisfies (b) trivially; one with `rate_limited` or `driven` dimensions does not, and its between-window traversals carry real cost (Rule 7.12.9).

**Rule 7.12.9 (run covering).** A *run covering* of a target scope (typically the conditions demanded in a set of environments whose `realization` is `physical`, Rule 7.7.3) is a set of feasible runs (Rule 7.12.8) whose `discharges` union equals the scope's verification procedures. The **computation of a minimal covering is an evaluation-engine concern** (Section 1.5): this specification defines the feasibility predicate a covering MUST satisfy, not the algorithm that finds it. Two structural forces drive the run count up: `driven` dimensions (Rule 4.5.1) with `reversible_within_run = false` (a dimension can be spent only once per run, so VPs requiring its fresh value compete), and the between-window traversal cost of `rate_limited` control (Rule 7.12.8(b)). Steering from one region to the next consumes real time and moves `driven` dimensions en route, so a covering trades window-packing against traversal expense. An environment whose constrained dimensions are all `settable`/`instantaneous` has near-zero traversal cost, and its covering degenerates toward one run (or unbounded parallel runs); an environment with `rate_limited` or `driven` dimensions does not, which is where the covering computation earns its keep. A covering computed by a run planner is a tool output (Rule 12.2.5).

-----

## 8. Composition Rules

Composition is the process by which independently verified child obligations combine to satisfy a parent obligation. Composition operates within a single declaration set (Sections 8.1–8.4) and across composed declaration sets (Section 8.5).

### 8.1 Basic Composition

**Rule 8.1.1.** A parent obligation O is *satisfied by composition* if and only if ALL of the following hold:

(a) Every child obligation Cᵢ of O with status `active` has status `satisfied` (every demanded condition of Cᵢ is discharged, Rule 9.3.4).

(b) The decomposition is *complete*: the set of child obligations covers all facets of the parent obligation (Rule 7.4.2).

(c) The decomposition is *consistent*: child constraints compose correctly to the parent constraint (Rule 7.4.1).

(d) The parent obligation O itself has every demanded condition discharged (Rule 9.3.4), resolved through its own satisfying design(s). Composition is necessary but not sufficient: child evidence demonstrates that parts work; parent-level evidence demonstrates that the composed whole exhibits the required behavior.

**Informative note:** When a parent obligation cannot yet be tested in the environments it demands (for example, an emergent property demanded only in the integrated-system environment, when the integrated system does not yet exist), the parent obligation has an evidence gap. The system remains at Level 1 (structurally conformant) until parent-level evidence is produced. The gap analysis machinery (Section 9) surfaces this as a standard evidence gap. Composition of child evidence does not substitute for parent evidence; it only demonstrates that the parts individually satisfy their constraints. The parent's evidence demonstrates the composed whole.

### 8.2 Cross-Shape Composition

**Rule 8.2.1.** A parent obligation MAY be decomposed into children satisfied by designs of different shapes. Each shape validates independently using its own constraint types and harnesses.

**Rule 8.2.2.** Completeness for cross-shape decomposition is checked at the parent obligation: does the set of designs, across all shapes, account for every facet of the parent obligation?

**Rule 8.2.3.** Consistency for cross-shape decomposition requires that shared parameters agree across shapes.

### 8.3 Composition Across Verification Environments

**Rule 8.3.1.** Evidence produced in environment E contributes to obligation O only if E is admitted for O (Rule 10.2.5), and discharges only the condition (O, π, E) at which it was produced. Evidence from one environment never substitutes for a condition demanded in another. OBSA defines no ranking of environments; no environment is acceptable by rank.

**Rule 8.3.2.** A parent obligation's evidence requirement is independent of its children's environments. The parent declares its own `applicable_environments` and requires evidence in those environments resolved through its own satisfying design. Child evidence in any environment does not substitute for parent-level evidence.

### 8.4 Monotonicity and Obligation Lifecycle

**Rule 8.4.1.** Active obligations are append-only. An obligation with status `active`, once declared, MUST NOT be removed from the declaration set. Its status may transition as follows:

- `active` → `superseded`: Replaced by a stricter obligation. The superseding obligation MUST be identified. All designs referencing the superseded obligation MUST be updated to reference the superseding obligation or re-verified.
- `active` → `withdrawn`: No longer applicable due to scope change. MUST include a rationale. MUST trigger re-evaluation of all parent obligations that depend on it. Safety obligations have additional restrictions (Rule 7.6.4).

Obligations with status `superseded` or `withdrawn` remain in the declaration graph for auditability but are excluded from gap analysis and verdict computation.

**Rule 8.4.2.** Evidence is append-only. Evidence, once produced, MUST NOT be removed. It may be superseded by more recent evidence.

**Rule 8.4.3.** Relaxation of an obligation constraint (making it less strict) MUST be accompanied by a rationale and MUST trigger re-evaluation of all designs that reference the obligation and all parent obligations that depend on it.

### 8.5 Composition Across Declaration Sets

When multiple declaration sets are composed into a single evaluation scope, obligations from one set may appear as children of obligations in another set. This creates multi-parent structure in the composed dependency graph.

**Rule 8.5.1.** Verdict propagation through compositional contribution is identical to verdict propagation through derivation parentage. If an obligation O appears in parent P's `children` list, O's satisfaction status contributes to P's composition verdict (Rule 8.1.1) regardless of whether O's derivation parent is P or another obligation in a different declaration set.

**Rule 8.5.2.** Constraint consistency (Rule 7.4.1) is checked for each parent-child pair independently. If obligation O is a child of both P₁ (in its own graph) and P₂ (in a cross-domain graph), O's constraints must be consistent with P₁'s constraints AND with P₂'s constraints. Consistency is per-pair, not collective.

**Rule 8.5.3.** Completeness (Rule 7.4.2) for a cross-domain parent is checked at composition time against the composed evaluation scope. A cross-domain parent's children may span multiple declaration sets. Completeness is assessed against the union of children across all composed sets. Missing children produce composition gaps (Section 9.1).

**Rule 8.5.4.** Acyclicity (Rule 7.1.4) is checked over the composed dependency graph. The composed graph includes both derivation edges (`parent` fields) and compositional edges (`children` lists across declaration sets). A cycle in the composed graph, even one that only exists through cross-set references, is a structural error.

-----

## 9. Gap Analysis

Gap analysis computes the difference between what should be verified and what has been verified.

### 9.1 Gap Types

|Gap Type      |Detection Rule                                                                                                |Nature                       |Resolution                          |Coverage Bucket|
|--------------|--------------------------------------------------------------------------------------------------------------|-----------------------------|------------------------------------|---------------|
|Decomposition |A requirement R has `decomposed_into` = empty                                                                 |Knowledge (ongoing)          |Analyze R and create obligations    |none (requirement-level)|
|Scenario      |A field observation, manufacturing nonconformance, or test failure cannot be mapped to any existing obligation|Knowledge (ongoing)          |Author a requirement with a named source (a `capa` record for an observed failure, or `fmea` for a newly identified failure mode) and decompose it|none|
|Infrastructure|An obligation O exists but no harness is registered for O's medium and no authored procedure covers the evidence. An unqualified harness still counts as available; its status is reported (Rule 12.2.5)|Tooling (one-time per medium)|Implement harness or author procedure|`no-evidence`|
|Observability |An obligation O exists but no TelemetryMapping exists for O's metric                                          |Tooling (one-time per metric)|Implement telemetry mapping         |`no-evidence`|
|Composition   |A requirement's `decomposed_into` or an obligation's `children` references an ID not resolvable in the current evaluation scope|Structural (boundary)|Resolve by composition or by authoring|none|
|Evidence Missing|A VP exists but no evidence entry references it                                                                                |Execution (one-time per VP)|Execute the verification procedure and capture evidence|`no-evidence`|
|Evidence Violated|Current evidence exists but the recomputed result is `fail` (Rule 9.3.2)                                                      |Remediation               |Fix the design or re-run verification after design change|`failed`|
|Evidence Stale  |Evidence exists but `component_revisions` do not match current component revisions                                             |Re-verification            |Re-execute the verification procedure against current component revisions|`stale`|
|Unimplemented Obligation|An active obligation has no satisfying design                                                                          |Design (ongoing)          |Author a design that satisfies the obligation, or resolve by composition|`no-design`|
|Underspecified / Unverifiable|Π(O) is empty, or O.`applicable_environments` is empty (Rule 9.3.1)                                               |Structural (obligation)   |Declare the obligation's applicable conditions, or list at least one verification environment|`underspecified` / `unverifiable`|
|Not Capable (Unverifiable)|A listed environment that declares structured capabilities is not capable for a condition (Rule 10.3.1)                     |Tooling (environment capability)|Extend the environment's declared control or instrumentation, or revise the obligation's `applicable_environments`|`unverifiable` (reason `not_capable`)|
|Self-report Mismatch|A supplied `result` disagrees with the recomputed result                                                                   |Integrity (verdict authority)|Investigate the producing procedure or tool; the recomputed result governs|Per the recomputation|
|Unrecorded Requirement|A Requirement declares no `record_id` (Rule 7.1.5)                                                                      |Knowledge (system of record)|Enter the requirement in the requirements system of record, or record why it stays outside|none (requirement-level)|
|Co-execution Conflict|A proposed or authored VerificationRun violates any of Rules 7.12.1–7.12.7 for its `discharges` set                        |Structural (run)          |Re-partition the run; add missing footprint declarations|none (run-level)|
|Batch Coverage|A run covering's discharged VP set does not equal its declared target scope                                                       |Structural (coverage)     |Add runs, or correct the scope; a VP claimed by no feasible run is surfaced|none (run-level)|
|Interference Discovery|The same obligation yields divergent verdicts across runs or across listed environments for the same plan entry and operating point (Rule 9.2.7)|Knowledge (feedback)|Localize the undeclared interference or the operating dimension one environment held fixed or did not model; promote to a Section 7.12 footprint declaration|none (run-level)|

**Conformance warning:** `derivable_but_linked`: evidence is manually linked to a VP that could have been derived through the harness/adapter pipeline. This is not a gap but a process compliance warning indicating that the automated path should be preferred. The preference for the automated path is conditional on the harness and adapter's qualification status, which is reported (Rule 12.2.5).

### 9.2 Gap Computation

Gap analysis operates only on obligations with status `active`.

**Rule 9.2.1.** For each requirement R: if R.decomposed_into is empty or all referenced obligations have status `withdrawn`, emit a decomposition gap for R. A requirement lacking a named external source is a structural violation (Rule 7.1.5), not a gap.

**Rule 9.2.2.** For each active obligation O, for each operating point π in Π(O), for each listed environment V in O.applicable_environments, for each entry of O's verification plan bound to V (Section 4.8.2): if no evidence exists at the satisfying design(s) for that (parameter, method, operating point, environment) tuple, emit an evidence gap. Classify the gap:

- If O has no satisfying design → Unimplemented Obligation gap (bucket `no-design`, Rule 9.3.4); the remaining classifications are not applied.
- If V declares structured capabilities and is not capable for (O, π, V) (Rule 10.3.1) → Not Capable gap (bucket `unverifiable`, reason `not_capable`, Rule 9.3.4); the remaining classifications are not applied.
- If no harness exists for O's domain AND no authored verification procedure covers the (parameter, method, operating point, environment) tuple → infrastructure gap.
- If no TelemetryMapping exists for O's metric → observability gap.
- Otherwise → Evidence Missing gap (evidence has not yet been produced but could be).

**Rule 9.2.3.** For each field observation, manufacturing nonconformance, or incident report: if the observation falls outside all obligation envelopes, emit a scenario gap. Resolution is a new or revised Requirement with a named source (Rule 7.1.5).

**Rule 9.2.4.** Runtime observations that satisfy an obligation's acceptance function, produced by a field environment (observation only, no control; all dimensions `observed`, Section 4.5), MAY be ingested as evidence for the operating points the observed system actually occupied, where that environment is listed for the obligation. Such evidence is subject to the same acceptance function evaluation and constraint type matching as design-time evidence.

**Rule 9.2.5.** For each requirement R: if R.decomposed_into references an obligation ID that does not exist in the current evaluation scope, emit a composition gap. For each obligation O: if O.children references an obligation ID that does not exist in the current evaluation scope, emit a composition gap. Classify by resolution path: if the referenced ID exists in a composed partner declaration set, the gap is resolvable by composition. If the referenced ID does not exist in any available declaration set, the gap is resolvable by authoring.

**Rule 9.2.6.** For each evidence entry E: if any component revision in E.component_revisions does not match the current revision of that component, emit an evidence staleness gap. Stale evidence is treated as `incomplete` for verdict computation; it does not count as passing evidence, but it is not deleted. When the verification procedure is re-executed against the current component revisions and new evidence is produced, the staleness gap is closed. The previous evidence remains in the record as historical data.

**Rule 9.2.7 (cross-run and cross-environment divergence).** For obligation O, operating point π and plan entry p, if current evidence from two runs, or from two environments both listed for O, with identical `component_revisions`, yields different recomputed results, the engine MUST emit an Interference Discovery gap. A divergence localizes to either (a) an undeclared interference in one of the runs, or (b) an operating dimension that one environment held fixed or did not model while the other drove it, which is a claim about that environment's model (Section 4.8.1). Classification is a domain judgment.

### 9.3 Coverage and Gap Metrics

**Rule 9.3.1 (demanded conditions).** C = ⋃ over active O of {(O, π, v) : π ∈ Π(O), v ∈ O.`applicable_environments`}, with two floor rules: (1) if Π(O) is empty, O contributes one `underspecified` condition; (2) if O.`applicable_environments` is empty, O contributes |Π(O)| `unverifiable` conditions. Hence |C(O)| ≥ 1 for every active obligation. C is computed from the active obligations, the declared operating conditions and the pinned corpus version alone; no design, verification plan, procedure, harness or adapter is consulted. |C| is invariant under changes to designs, plans, harnesses and adapters.

**Rule 9.3.2 (verdict authority).** The evaluation engine MUST compute each evidence result by applying the constraint type's acceptance function to `measurement` against the constraint value current in the evaluated declaration set (Section 6.1). A supplied `result` is a claim, not a verdict: where it differs from the computed value, the computed value governs and the discrepancy MUST be reported (Rule 12.2.4). Evidence with no evaluable measurement has result `inconclusive` and discharges nothing. For `inspection` and `demonstration` entries, a recorded judgment by a named, accountable person against the declared constraint (who, role, date, what was judged) is an evaluable measurement. Where a measurement is a bare boolean emitted by the procedure under evaluation, the report counts the record as a self-attested measurement.

**Rule 9.3.3 (oracle independence).** Evidence discharges a condition only through an acceptance computed under Rule 9.3.2 against a declared constraint: the obligation's own, or a design constraint that refines it (Rule 7.3), on an obligation that traces to a Requirement satisfying Rule 7.1.5. An assertion internal to a test, or a pass criterion that is not a declared constraint, is never an oracle. A recorded human judgment under Rule 9.3.2 is evaluated against the declared constraint and is not such an assertion. Test size and element count are not criteria: a single-element test of an intrinsic claim is evidence when this rule holds.

**Rule 9.3.4 (coverage partition).** For c = (O, π, v), the demanded plan entries are those of O's verification plan bound to v (Section 4.8.2). Each c ∈ C is placed in exactly one bucket: the first that applies in this order: (1) `underspecified` (floor rule 1); (2) `unverifiable` (floor rule 2: no environment listed); (3) `no-design` (O has no satisfying design); (4) `failed` (current admitted evidence for some demanded entry fails on recomputation); (4a) `unverifiable`, reason `not_capable` (v declares structured capabilities and is not capable for c, Rule 10.3.1); (5) `undecidable` (some demanded entry's evaluation is deferred or exceeds the engine bound: Section 1.6, Section 5.3); (6) `no-evidence` (some demanded entry has no admitted evidence at (π, v), including unevaluable or `inconclusive` evidence and conjunctions not recorded); (7) `stale` (some demanded entry has only stale evidence, passing or failing); (8) `provisional` (otherwise passing, and O declares `provisional_closure`); (9) `discharged`. Bucket counts sum to |C|. Two methods for one parameter count once; one evidence record may discharge several conditions. No bucket other than (1)–(3) ranks above `failed`.

**Rule 9.3.5 (coverage).** Coverage = |discharged| / |C|, condition-weighted, stated against a named coverage query over a pinned corpus version. A coverage query is declared as `{name, corpus_version, select}`. `corpus_version` identifies the declaration-set revision and the version of every external input read (Section 1.6). `select` is one of: `all` (every active obligation); an explicit list of Requirement IDs; or a filter on Requirement `source_type` and `record_id` prefix. The query's obligations are the active obligations reachable downward from the selected Requirements. C is restricted to them. Every coverage figure is reported with its query name and corpus version. The default query is `all` over the evaluated declaration set.

**Rule 9.3.6 (single pass).** The condition-level gap report is {c ∈ C : bucket(c) ≠ discharged}, grouped by bucket and classified per Section 9.1, computed from the same partition as coverage. Requirement-level and run-level gaps are reported alongside it.

**Rule 9.3.7 (metric identity).** In this revision metric identity is lexical: a plan parameter, telemetry mapping or observable metric matches a constraint metric if it is equal to it or names it through `maps_to`. A mismatch never removes a condition from C. This governs Rules 7.3, 7.4.1, 7.5.1, 7.9.3 and 10.3.1(b).

The following metrics are computed from gap analysis:

- **Obligation completion (secondary):** (active obligations with every demanded condition discharged) / (active obligations)
- **Evidence freshness:** (evidence entries with current component revisions) / (total evidence entries)
- **Decomposition completeness:** (requirements with ≥ 1 active obligation) / (total requirements)
- **Infrastructure readiness:** (active obligations with available harness or authored verification procedures covering all verification plan entries) / (total active obligations)
- **Observability readiness:** (active obligations with TelemetryMapping) / (total active obligations)

-----

## 10. Verification Environments

### 10.1 Environments Are Not Ordered

OBSA defines no ordering among verification environments. Environment names are instance labels. Whether evidence from an environment is admitted is determined by Section 10.2.

**Rule 10.1.1.** [Deleted; replaced by explicit `applicable_environments` binding. Evidence acceptance is determined by whether evidence was produced at a verification environment listed in the obligation's `applicable_environments`, not by fidelity ordering.]

**Rule 10.1.2.** [Withdrawn in 0.2.2: no hierarchy exists. Environment variants are distinct VerificationEnvironment instances.]

### 10.2 Evidence Admission

**Rule 10.2.1.** [Withdrawn in 0.2.2 with `min_fidelity`. Where evidence is demanded is stated by `applicable_environments` and plan-entry bindings (Rules 7.7.2, 7.7.7); admission by Rule 10.2.5; safety guidance by Rule 7.6.5.]

**Rule 10.2.2.** [Withdrawn in 0.2.2: duplicate of Rule 7.6.3; see Rule 7.6.5.]

**Rule 10.2.3.** [Consolidated into Rule 7.7.3 and Rule 9.3.1. The verification matrix for a release is C under the release's declared coverage query.]

**Rule 10.2.4.** [Withdrawn in 0.2.2: no ordering to check. See Rules 10.2.5 and 10.3.1.]

**Rule 10.2.5 (interim admission).** Evidence produced in environment v is admitted for obligation O only if v ∈ O.`applicable_environments`. No ordering among environments exists or is implied. Evidence from an environment that is not admitted never discharges a condition.

**Rule 10.2.6 (admission disclosure).** A conformance report MUST state the admission basis it applied. For this revision: admission by authored listing (Rule 10.2.5); declared capability, where structured, is evaluated for classification only (Rules 9.3.4, 10.3.1); agreement of an environment's verdicts with reality (qualification) is not evaluated.

### 10.3 applicable_environments and Environment Capability

`applicable_environments` is the obligation-side statement of where evidence is demanded. It is authored at all levels. Environment capability never removes entries from it or changes C. Structured environment capabilities (Section 4.5) validate it and may propose additions (Rule 10.3.1).

| Level | State |
|-------|-------|
| Level 1 | Authored; unstructured capabilities |
| Level 2 | Authored; validated against structured capabilities where declared (Rule 10.3.1) |
| Level 3 | Authored; engine proposals reported, never applied (Rule 10.3.1) |

Section 4.5 supplies the structured environment capabilities, enabling the Level 2 validation of the table above.

**Rule 10.3.1 (environment capability).** A verification environment E that declares structured capabilities (Section 4.5) is *capable* for a verification procedure VP if and only if (a) every dimension VP's required region constrains has a DimensionControl declared by E (`settable`, `driven`, or `observed`) under which the region is reachable (Rule 7.12.8(b)), and (b) E's `observable_metrics` cover VP's evidence schema (Rule 9.3.7). Capability is never inferred from a dimension's `kind`. E is capable for a condition (O, π, E) if it is capable for every procedure that realizes, at π in E, a plan entry demanded at that condition. Where O has no satisfying design or no such procedure exists, capability is not evaluated for that condition. Conditions for which E is not capable are bucketed `unverifiable` with reason `not_capable`, below `failed` (Rule 9.3.4). Where the only cause is a metric-name mismatch in (b), the report says so. An environment that declares no structured capabilities is not evaluated. The engine MAY propose environments that are capable but unlisted; proposals are informative. Capability does not affect admission (Rule 10.2.5) and is evaluated only after `failed`, so a failing measurement from a listed environment is never masked. Capability is necessary for a condition to be discharged in E and is not evidence that the environment's verdicts are correct.

**Informative note: run routing.** Every environment listed in an obligation's `applicable_environments` is demanded (Rule 7.7.2), so declared `cost` (Section 4.5) never selects which listed environment is demanded and never lets one listed environment stand in for another. A run planner may use `cost` only to order or batch runs among the demanded (obligation, point, environment) conditions, for example by scheduling cheaper runs first. Routing is not a conformance requirement.

-----
## 11. Extensibility

### 11.1 Extension Categories

OBSA has three categories of extensibility:

|Category          |Governance                                                                |Examples                                                                                                                                                                                     |
|------------------|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Fixed core        |Not extensible                                                            |Declaration hierarchy (Requirement → Obligation → Design), structural rules, composition rules, gap types, coverage partition (Section 9.3), verification methods (TIAD), evidence-through-design model, structural parametricity invariant (Section 3.1). The co-execution relations (Section 7.12), the run gap types (Section 9.1) and the VerificationRun derived type (Section 4.7.8) are part of the fixed core. They were added by core revision 0.3.0, not by governed extension, are domain-invariant (Rule 11.2.2), and are therefore subject to Rule 11.3.3, not to Sections 5.6 and 6.3|
|Governed extension|Extensible with documented justification per rules in Sections 5.6 and 6.3|Design shapes, constraint types                                                                                                                                                              |
|Free instantiation|Freely extensible by any user                                             |Requirements, obligations, designs, operating conditions, operating dimensions, components, verification environments, harnesses, adapters, authored verification procedures, telemetry mappings. Harnesses and adapters are tools; their qualification status is reported (Rule 12.2.5). Operating dimensions (Section 4.4a) are declared freely; the structural rules that consume them are fixed core|

### 11.2 Domain Extension

**Rule 11.2.1.** Extending OBSA to a new domain requires harnesses and adapters for that domain's interface media and verification tools for derived verification. Verification tools for derived verification report their qualification status (Rule 12.2.5); until qualified, their outputs are disclosed as unqualified. Alternatively, authored verification procedures may be declared directly for domains where harnesses have not yet been implemented. Authored procedures provide immediate verification coverage; harnesses systematize and automate what authored procedures do manually. The declaration hierarchy, structural rules, composition rules, and gap analysis are domain-invariant. This is a consequence of structural parametricity (Section 3.1).

**Rule 11.2.2.** A domain extension MUST NOT modify the structural rules, composition rules, or conformance criteria. Any proposed modification that would be required for a new domain indicates a structural parametricity violation (Section 3.1) in the existing rule.

### 11.3 Backward Compatibility

**Rule 11.3.1.** Adding a new shape or constraint type MUST NOT invalidate any existing conformant declaration set.

**Rule 11.3.2.** New shapes and constraint types are additive. Existing declarations continue to be valid under their original shapes and constraint types.

**Rule 11.3.3 (core revisions).** Changes to the fixed core (Section 11.1) are made only by numbered core revision, recorded in the Revision History with the affected rules. A core revision is neither a governed extension (Sections 5.6 and 6.3) nor a domain extension (Section 11.2).

**Rule 11.3.4 (deprecation and severity introduction).** A core revision that changes or removes an instance-type field MUST accept the previous form as deprecated for one minor version, with a warning, and MUST list it in Annex C. A core revision that adds a field MUST NOT make it required in the same minor version. A core revision that adds a requirement which a declaration set conformant to the previous revision could violate MUST report violations as warnings for one minor version and MUST list the rule in Annex C.

### 11.4 Configuration Management

Each declaration set represents a single system configuration. Multi-configuration management (e.g., fleet variants, software versions coexisting in production) is handled at the instantiation layer through standard version control and configuration management practices. See Section 1.4; multiple configurations are assured by composing or comparing declaration sets (Section 8.5).

-----

## 12. Conformance

### 12.1 Conformance Levels

OBSA defines three conformance levels:

**Level 1: Structurally Conformant.** A declaration set is structurally conformant if and only if all rules in Section 7, plus Rules 4.4.1, 4.4.2 and 4.5.1 and the well-formedness of authored VerificationRuns (Section 4.7.8), are satisfied. The declaration set is well-formed. Designs refine obligations. Obligations trace to requirements. Requirements trace to named external sources (Rule 7.1.5). The DAG is acyclic. Cross-obligation consistency holds. Verification plans are complete. State-dependent obligation activation is consistent (Rule 7.10). A declaration set that includes authored VerificationRuns MUST satisfy the co-execution rules of Section 7.12 for each run; this is checkable from the declaration set alone, without executing anything. This level can be checked mechanically by an evaluation engine whose qualification status is reported (Rule 12.2.5), from the declaration set alone, without executing any tests.

**Level 2: Evidence-Evaluated.** A declaration set is evidence-evaluated if and only if:
(a) It is structurally conformant (Level 1).
(b) At least one evidence entry exists, referencing a valid VP and design.
(c) All existing evidence has been evaluated by recomputation (Rule 9.3.2) through constraint type acceptance functions and assigned a verdict.
(d) Evidence freshness has been checked: all evidence entries have current `component_revisions` (no staleness gaps).
(e) The coverage partition (Rule 9.3.4) has been computed.
(f) Where verification environments declare structured capabilities, `applicable_environments` has been validated against Rule 10.3.1.
Run coverings and Batch Coverage gaps are computed at Level 2 when footprints are present. This level represents a declaration set where evidence exists and has been mechanically evaluated, but may not yet cover all obligations. Gap analysis (Section 9) reports exactly what is missing. This is the operational level for CI: every merge produces a conformance report showing current verdicts and remaining gaps.

**Level 3: Evidence-Complete.** A declaration set is evidence-complete if and only if:
(a) It is evidence-evaluated (Level 2).
(b) Every demanded condition (Rule 9.3.1) is `discharged` (Rule 9.3.4). For state-specific obligations (Rule 7.10), evidence is required only under the operating conditions applicable during the obligation's mapped states.
(c) All composition rules (Section 8) are satisfied for every parent obligation, including parent-level evidence (Rule 8.1.1(d)).
(d) No decomposition gaps exist (every requirement has at least one active obligation).

Conformance levels are not merely descriptive; they condition downstream operations. See Section 1.6 (Gated execution) for the requirement that deployment and release artifacts are gated on conformance.

### 12.2 Conformance Checking

**Rule 12.2.1.** Level 1 conformance is checkable from the declaration set alone. No test execution required. This includes verification plan coverage checking (Rule 7.9) and state-dependent obligation consistency checking (Rule 7.10).

**Rule 12.2.2.** Level 2 conformance requires the declaration set plus at least one evidence entry. The evaluation engine (Rule 12.2.5) evaluates all existing evidence by recomputation (Rule 9.3.2) through acceptance functions, checks component revision freshness, and computes verdicts for all obligations with evidence.

**Rule 12.2.3.** Level 3 conformance requires the declaration set plus a complete evidence set. The evidence set is compared against the demanded condition set C (Rule 9.3.1). Evidence is located at the design level and evaluated through constraint type acceptance functions.

**Rule 12.2.4.** A conformance report MUST enumerate:

- All structural rule violations (Level 1 failures).
- All verification plan coverage gaps (obligation parameters not covered by design verification plans).
- All evidence gaps with classification (Level 2/3 gaps).
- All staleness gaps with affected component revisions.
- All composition failures with specific parent obligations and failing conditions.
- All state-dependent obligation consistency violations (Rule 7.10 failures).
- Coverage metrics per Section 9.3.
- The coverage partition: counts per bucket, overall and per obligation.
- Self-report mismatches (Rule 9.3.2).
- The admission basis (Rule 10.2.6).
- Tool qualification status (Rule 12.2.5).
- Warnings under Rule 11.3.4, Rule 7.6.5 warnings, and Unrecorded Requirement gaps.
- Safety obligations using a declared operating-point subset (Rule 7.6.2).
- Self-attested measurements (Rule 9.3.2).
- The coverage query name and corpus version of every coverage figure (Rule 9.3.5).
- Run violations and Co-execution Conflict, Batch Coverage and Interference Discovery gaps.

**Rule 12.2.5 (tool qualification status).** A conformance report MUST state the qualification status of every tool that produced it or produced or computed values it evaluates: evaluation engine, harnesses, adapters, run planners. A tool's own outputs never constitute its qualification. Its qualification evidence MUST NOT be produced or judged by that tool, or by any tool whose own qualification depends, directly or transitively, on that tool. Dependencies among tool qualifications MUST form an acyclic graph. Whether a tool's qualification holds is never computed by that tool. This revision defines no qualification mechanism; until one is defined, the status of every tool is `unqualified`.

### 12.3 Partial Conformance

In practice, systems achieve conformance incrementally. A declaration set may be structurally conformant (Level 1), progress to evidence-evaluated (Level 2) as evidence is produced, and work toward evidence-complete (Level 3). Gap analysis (Section 9) provides the roadmap from Level 1 through Level 2 to Level 3.

-----

## 13. Examples

### 13.1 Cross-Domain Composition Example

This example demonstrates an obligation at level `system` decomposing through multiple shapes, domains, and verification environments for a transport aircraft one-engine-inoperative (OEI) continued takeoff scenario.

**Requirement:**

```yaml
id: REQ-SAFE-001
title: Single Engine Continued Takeoff
source_type: hazard_analysis
source_reference: FHA-2025-003, Hazard H7; 14 CFR 25.121(a)
record_id: REQ-SAFE-001
text: "The aircraft shall maintain a positive climb gradient of not less
       than 2.4% following a single engine failure recognized at or above
       V1 during takeoff with landing gear retracted."
# decomposed_into: computed from obligation parent fields (Rule 7.11)
```

**System obligation:**

```yaml
id: OBL-SYS-OEI-CLIMB
title: OEI Climb Gradient at Second Segment
parent: REQ-SAFE-001
source_type: requirement_decomposition
scope: emergent
elements: [COMP-AIRFRAME, COMP-ENGINE-L, COMP-ENGINE-R, COMP-FLIGHT-COMPUTER]
level: system
status: active
constraint_type: CT-THRESHOLD
constraint:
  metric: oei_second_segment_climb_gradient
  operator: ">="
  value: 2.4
  unit: percent
verification_plan:
  - parameter: oei_second_segment_climb_gradient
    method: test
    rationale: "Flight test measurement under OEI conditions"
    environments: [VE-FLIGHT-TEST]
  - parameter: aerodynamic_model_correlation
    method: analysis
    rationale: "CFD model validated against wind tunnel and flight test"
    environments: [VE-INTEGRATED-TEST]
  - parameter: pilot_engine_failure_recognition
    method: demonstration
    rationale: "Witnessed simulator evaluation per AC 25-7D"
    environments: [VE-INTEGRATED-TEST]
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-INTEGRATED-TEST
  - VE-FLIGHT-TEST
# children: computed from scanning parent fields (Rule 7.11)
# satisfied_by: computed from scanning design satisfies fields (Rule 7.11)
```

**Subsystem obligations (five children, different shapes):**

```yaml
# Child 1: Left engine available thrust as the remaining engine (Envelope shape)
id: OBL-SUB-ENGINE-L-OEI-THRUST
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: intrinsic
elements: [COMP-ENGINE-L]
level: subsystem
status: active
constraint_type: CT-THRESHOLD
constraint:
  metric: max_takeoff_thrust_per_engine
  operator: ">="
  value: 105
  unit: kN
verification_plan:
  - parameter: max_takeoff_thrust_per_engine
    method: test
    rationale: "Dynamometer measurement across temperature and altitude range"
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-DYNO-CELL
# satisfied_by: computed (Rule 7.11)

# Child 2: Right engine available thrust as the remaining engine (Envelope shape)
id: OBL-SUB-ENGINE-R-OEI-THRUST
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: intrinsic
elements: [COMP-ENGINE-R]
level: subsystem
status: active
constraint_type: CT-THRESHOLD
constraint:
  metric: max_takeoff_thrust_per_engine
  operator: ">="
  value: 105
  unit: kN
verification_plan:
  - parameter: max_takeoff_thrust_per_engine
    method: test
    rationale: "Dynamometer measurement across temperature and altitude range"
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-DYNO-CELL
# satisfied_by: computed (Rule 7.11)

# Child 3: Aircraft drag budget in clean configuration (Budget shape)
id: OBL-SUB-DRAG-CLEAN-CONFIG
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: emergent    # emergent, satisfied by a Budget design
elements: [COMP-AIRFRAME, COMP-ENGINE-L, COMP-ENGINE-R, COMP-LANDING-GEAR]
level: subsystem
status: active
constraint_type: CT-BALANCE
constraint:
  metric: net_thrust_minus_drag
  operator: ">="
  value: 0
  unit: kN
verification_plan:
  - parameter: net_thrust_minus_drag
    method: analysis
    rationale: "Drag accounting validated against wind tunnel data"
  - parameter: component_drag_allocations
    method: inspection
    rationale: "Review of drag buildup document against ICD"
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-CFD
# satisfied_by: computed (Rule 7.11)

# Child 4: Flight control asymmetric thrust compensation (Exchange shape)
id: OBL-SUB-FCS-ASYMMETRY-COMP
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: emergent    # compensation depends on both channels jointly; not reducible to one pair
elements: [COMP-FLIGHT-COMPUTER, COMP-FADEC-L, COMP-FADEC-R]
level: subsystem
status: active
constraint_type: CT-TIMING
constraint:
  metric: asymmetry_compensation_time
  operator: "<="
  value: 3
  unit: s
verification_plan:
  - parameter: asymmetry_compensation_time
    method: test
    rationale: "Measured response on the iron-bird rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case execution time analysis"
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-IRONBIRD-RIG
# satisfied_by: computed (Rule 7.11)

# Child 5: FADEC failure detection and engine reconfiguration (Exchange shape)
id: OBL-SUB-FADEC-RECONFIG
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: relational
elements: [COMP-FADEC-L, COMP-FADEC-R]
level: subsystem
status: active
constraint_type: CT-TIMING
constraint:
  metric: failure_detect_to_reconfig
  operator: "<="
  value: 1
  unit: s
verification_plan:
  - parameter: failure_detect_to_reconfig
    method: test
    rationale: "Fault injection test on the iron-bird rig"
applicable_conditions:
  - OC-ALTITUDE-TAKEOFF
  - OC-WEIGHT-MTOW
  - OC-TEMPERATURE-ISA-PLUS-20
applicable_environments:
  - VE-IRONBIRD-RIG
# satisfied_by: computed (Rule 7.11)
```

**Components (grounded references):**

```yaml
id: COMP-ENGINE-L
title: Left Engine
source:
  resolution_protocol: part_number
  reference: "PW1133G-JM"
  qualifier: "serial_range:2025+"

id: COMP-ENGINE-R
title: Right Engine
source:
  resolution_protocol: part_number
  reference: "PW1133G-JM"
  qualifier: "serial_range:2025+"

id: COMP-FLIGHT-COMPUTER
title: Primary Flight Computer
source:
  resolution_protocol: part_number
  reference: "FCC-4200A"

id: COMP-FADEC-L
title: Left Engine FADEC
source:
  resolution_protocol: part_number
  reference: "FADEC-EEC-900"
  qualifier: "channel:A"

id: COMP-FADEC-R
title: Right Engine FADEC
source:
  resolution_protocol: part_number
  reference: "FADEC-EEC-900"
  qualifier: "channel:B"

id: COMP-AIRFRAME
title: Airframe Structure
source:
  resolution_protocol: drawing_number
  reference: "DWG-100-001"
  qualifier: "config:clean"

id: COMP-LANDING-GEAR
title: Landing Gear Assembly
source:
  resolution_protocol: drawing_number
  reference: "DWG-320-001"
```

**Operating dimensions, conditions and environments:**

```yaml
id: DIM-PRESSURE-ALTITUDE
name: pressure_altitude
unit: ft
kind: exogenous

id: DIM-GROSS-WEIGHT
name: gross_weight
unit: kg
kind: exogenous

id: DIM-AMBIENT-TEMPERATURE-DEVIATION
name: ambient_temperature_deviation
unit: degC
kind: exogenous

id: OC-ALTITUDE-TAKEOFF
title: Takeoff Field Elevation Range
parameter: pressure_altitude
range: {min: 0, max: 8000, unit: ft}
severity: nominal

id: OC-WEIGHT-MTOW
title: Maximum Takeoff Weight
parameter: gross_weight
range: {min: 85000, max: 85000, unit: kg}
severity: nominal

id: OC-TEMPERATURE-ISA-PLUS-20
title: Hot Day, ISA+20
parameter: ambient_temperature_deviation
range: {min: 20, max: 20, unit: degC}
severity: nominal

id: VE-DYNO-CELL
title: Engine Dynamometer Cell
description: "Engine test cell with thrust measurement and altitude and temperature simulation"
realization: physical
controllable_dimensions:
  - {dimension: pressure_altitude, controllability: settable, mode: rate_limited}
  - {dimension: ambient_temperature_deviation, controllability: settable, mode: rate_limited}
  # gross weight is applied as the takeoff thrust-demand schedule for that weight
  - {dimension: gross_weight, controllability: settable, mode: instantaneous}
observable_metrics: [max_takeoff_thrust_per_engine]

id: VE-CFD
title: Aerodynamic CFD Simulation
description: "Computational fluid dynamics drag model correlated with wind tunnel data"
realization: simulated
controllable_dimensions:
  - {dimension: pressure_altitude, controllability: settable, mode: instantaneous}
  - {dimension: gross_weight, controllability: settable, mode: instantaneous}
  - {dimension: ambient_temperature_deviation, controllability: settable, mode: instantaneous}
observable_metrics: [net_thrust_minus_drag, component_drag_allocations]

id: VE-IRONBIRD-RIG
title: Flight Control Iron-Bird Rig
description: "Flight control computer and FADEC hardware with simulated engines and aircraft dynamics"
realization: mixed
controllable_dimensions:
  # set as inputs to the simulated engine and aircraft models
  - {dimension: pressure_altitude, controllability: settable, mode: instantaneous}
  - {dimension: gross_weight, controllability: settable, mode: instantaneous}
  - {dimension: ambient_temperature_deviation, controllability: settable, mode: instantaneous}
observable_metrics: [asymmetry_compensation_time, failure_detect_to_reconfig]

id: VE-INTEGRATED-TEST
title: Integrated Aircraft Test
description: "Integrated aircraft systems with a validated performance model and a piloted engineering simulator"
realization: mixed
controllable_dimensions:
  # set as inputs to the performance model and the simulator
  - {dimension: pressure_altitude, controllability: settable, mode: instantaneous}
  - {dimension: gross_weight, controllability: settable, mode: instantaneous}
  - {dimension: ambient_temperature_deviation, controllability: settable, mode: instantaneous}
observable_metrics: [oei_second_segment_climb_gradient, aerodynamic_model_correlation,
                     pilot_engine_failure_recognition]

id: VE-FLIGHT-TEST
title: Flight Test
description: "Instrumented test aircraft in flight"
realization: physical
controllable_dimensions:
  # no control authority: test points are flown when conditions occur
  - {dimension: pressure_altitude, controllability: observed}
  - {dimension: gross_weight, controllability: observed}
  - {dimension: ambient_temperature_deviation, controllability: observed}
observable_metrics: [oei_second_segment_climb_gradient]
```

**Designs (five designs, three shapes):**

```yaml
# Envelope design: engine thrust characterization
id: DES-ENV-ENGINE-THRUST
shape: envelope
title: Engine Thrust Envelope Characterization
satisfies: [OBL-SUB-ENGINE-L-OEI-THRUST, OBL-SUB-ENGINE-R-OEI-THRUST]
components: [COMP-ENGINE-L, COMP-ENGINE-R]
dimension: max_takeoff_thrust
validated_range: {min: 0, max: 118, unit: kN}
constraints:
  - {type: CT-THRESHOLD, metric: max_takeoff_thrust_per_engine,
     operator: ">=", value: 118, unit: kN}
  # 118 kN design > 105 kN obligation: refinement satisfied.
verification_plan:
  - parameter: max_takeoff_thrust_per_engine
    method: test
    rationale: "Dynamometer test across altitude/temperature matrix"

# Budget design: drag accounting
id: DES-BDG-DRAG
shape: budget
title: OEI Drag Budget
satisfies: [OBL-SUB-DRAG-CLEAN-CONFIG]
components: [COMP-AIRFRAME, COMP-ENGINE-L, COMP-ENGINE-R, COMP-LANDING-GEAR]
resource: aerodynamic_drag
unit: kN
supply: 118    # single engine available thrust at MTOW, ISA+20
allocations:
  - {component: COMP-AIRFRAME, demand: 85, note: "clean config at V2, MTOW"}
  - {component: COMP-ENGINE-L, demand: 8, note: "windmilling drag, failed engine"}
  - {component: COMP-LANDING-GEAR, demand: 0, note: "retracted"}
margin_policy: {minimum_margin_percent: 20, rationale: "climb gradient margin"}
constraints:
  - {type: CT-BALANCE, metric: net_thrust_minus_drag,
     operator: ">=", value: 25, unit: kN}
  # 25 kN net > 0 kN obligation: refinement satisfied.
verification_plan:
  - parameter: net_thrust_minus_drag
    method: analysis
    rationale: "CFD drag buildup validated against wind tunnel"
  - parameter: component_drag_allocations
    method: inspection
    rationale: "Drag buildup spreadsheet review against ICD"

# Exchange design: flight control ↔ FADEC asymmetry compensation
id: DES-EXCH-FCS-FADEC
shape: exchange
title: FCS-FADEC Asymmetric Thrust Compensation
satisfies: [OBL-SUB-FCS-ASYMMETRY-COMP]
components: [COMP-FLIGHT-COMPUTER, COMP-FADEC-L, COMP-FADEC-R]
medium: ARINC_429
flows:
  - name: thrust_asymmetry_command
    from: COMP-FLIGHT-COMPUTER
    to: COMP-FADEC-L
    protocol: ARINC_429
    encoding: label_312
    timing: "50 Hz"
  - name: engine_status
    from: COMP-FADEC-L
    to: COMP-FLIGHT-COMPUTER
    protocol: ARINC_429
    encoding: label_200
    timing: "20 Hz"
  - name: thrust_asymmetry_command
    from: COMP-FLIGHT-COMPUTER
    to: COMP-FADEC-R
    protocol: ARINC_429
    encoding: label_312
    timing: "50 Hz"
  - name: engine_status
    from: COMP-FADEC-R
    to: COMP-FLIGHT-COMPUTER
    protocol: ARINC_429
    encoding: label_200
    timing: "20 Hz"
constraints:
  - {type: CT-TIMING, metric: asymmetry_compensation_time,
     operator: "<=", value: 2.5, unit: s}
  # 2.5s design < 3s obligation: refinement satisfied.
verification_plan:
  - parameter: asymmetry_compensation_time
    method: test
    rationale: "ARINC 429 bus timing measurement on the iron-bird rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case execution time analysis of FCS task scheduling"

# Exchange design: FADEC internal failure detection and reconfiguration
id: DES-EXCH-FADEC-CMD
shape: exchange
title: FADEC Cross-Channel Failure Detection
satisfies: [OBL-SUB-FADEC-RECONFIG]
components: [COMP-FADEC-L, COMP-FADEC-R]
medium: CCDL
# dedicated point-to-point link: relational (Section 4.2). Over a shared network, detection timing would be emergent, naming the network's participants.
flows:
  - name: channel_health_status
    from: COMP-FADEC-L
    to: COMP-FADEC-R
    protocol: CCDL
    encoding: health_word
    timing: "10 ms period"
  - name: channel_health_status
    from: COMP-FADEC-R
    to: COMP-FADEC-L
    protocol: CCDL
    encoding: health_word
    timing: "10 ms period"
constraints:
  - {type: CT-TIMING, metric: failure_detect_to_reconfig,
     operator: "<=", value: 0.8, unit: s}
  # 0.8s design < 1s obligation: refinement satisfied.
verification_plan:
  - parameter: failure_detect_to_reconfig
    method: test
    rationale: "Fault injection test on the iron-bird rig over the cross-channel data link"

# Envelope design: composed aircraft OEI climb performance
id: DES-ENV-OEI-CLIMB
shape: envelope
title: OEI Second-Segment Climb Performance
satisfies: [OBL-SYS-OEI-CLIMB]
components: [COMP-AIRFRAME, COMP-ENGINE-L, COMP-ENGINE-R, COMP-FLIGHT-COMPUTER]
dimension: oei_second_segment_climb_gradient
validated_range: {min: 2.9, max: 2.9, unit: percent}
constraints:
  - {type: CT-THRESHOLD, metric: oei_second_segment_climb_gradient,
     operator: ">=", value: 2.9, unit: percent}
  # 2.9% design > 2.4% obligation: refinement satisfied.
verification_plan:
  - parameter: oei_second_segment_climb_gradient
    method: test
    rationale: "Flight test measurement of climb gradient with one engine inoperative"
  - parameter: aerodynamic_model_correlation
    method: analysis
    rationale: "Performance model correlated against wind tunnel and flight test data"
  - parameter: pilot_engine_failure_recognition
    method: demonstration
    rationale: "Witnessed piloted simulator evaluation per AC 25-7D"
```

**Composition check:**

- **Refinement:** All five designs have constraints stricter than their obligations. ✓
  - OEI climb gradient: 2.9% ≥ 2.4%
  - Engine thrust: 118 kN ≥ 105 kN
  - Net thrust: 25 kN ≥ 0 kN (actual gradient ~2.9% > 2.4%)
  - Asymmetry compensation: 2.5s ≤ 3s
  - FADEC reconfiguration: 0.8s ≤ 1s
- **Verification plan coverage:** All obligation verification plan parameters are covered by corresponding design verification plan entries with matching methods. ✓
- **Consistency:** Thrust (118 kN) minus drag (93 kN) = 25 kN net, yielding climb gradient above 2.4% at MTOW. Budget closes. ✓
- **Completeness:** All contributing factors covered (five children): available thrust, drag accounting, flight control response, engine reconfiguration. ✓
- **Environments:** each obligation's demanded conditions are discharged by evidence from its own listed environments (Rule 10.2.5); child evidence does not substitute for the parent's (Rule 8.3.2). The parent demands flight-test evidence in VE-FLIGHT-TEST and analysis and demonstration evidence in VE-INTEGRATED-TEST, per its plan-entry bindings (Rule 7.7.7), through its own satisfying design DES-ENV-OEI-CLIMB. ✓
- **Explicit conditions:** Each obligation declares its applicable conditions (Rule 7.7.1) and applicable environments (Rule 7.7.2). Every obligation descends from REQ-SAFE-001 and is a safety obligation, so each lists all three declared conditions (Rule 7.6.2). ✓
- **Physical anchoring:** Each safety obligation lists an environment whose `realization` is `physical` or `mixed` for the metric it constrains (Rule 7.6.5), except OBL-SUB-DRAG-CLEAN-CONFIG, whose only listed environment, VE-CFD, is `simulated`. The engine reports a Rule 7.6.5 warning for `net_thrust_minus_drag`, and the conformance report lists it (Rule 12.2.4). The drag figures are anchored by the wind-tunnel correlation stated in its verification plan rationale, and the composed result is anchored by the parent's own evidence in VE-FLIGHT-TEST.
- **Computed fields:** `children`, `satisfied_by`, and `decomposed_into` are not authored; they are computed by the evaluation engine from the `parent` and `satisfies` references (Rule 7.11). ✓

**Demanded conditions:** Each obligation lists three conditions on three different parameters, so it has exactly one operating point (Rule 7.7.5). Its demanded conditions are that point crossed with its listed environments (Rule 9.3.1).

|Obligation                 |\|Π(O)\||\|V(O)\||\|C(O)\||
|---------------------------|-------|-------|-------|
|OBL-SYS-OEI-CLIMB          |1      |2      |2      |
|OBL-SUB-ENGINE-L-OEI-THRUST|1      |1      |1      |
|OBL-SUB-ENGINE-R-OEI-THRUST|1      |1      |1      |
|OBL-SUB-DRAG-CLEAN-CONFIG  |1      |1      |1      |
|OBL-SUB-FCS-ASYMMETRY-COMP |1      |1      |1      |
|OBL-SUB-FADEC-RECONFIG     |1      |1      |1      |

**Result:** OBL-SYS-OEI-CLIMB is satisfied by composition if: (1) all five children have every demanded condition discharged in their listed environments under all applicable operating conditions, resolved through their respective designs; AND (2) the parent obligation itself has every demanded condition discharged in its own listed environments, resolved through its own satisfying design, demonstrating the actual exhibited climb gradient under OEI conditions.

-----

## 14. References

- Benveniste, A., et al. (2018). Contracts for System Design. Foundations and Trends in Electronic Design Automation, 12(2-3), 124-400.
- INCOSE. (2023). Systems Engineering Handbook, version 5.0.
- INCOSE. (2022). Guide to Writing Requirements, version 3.1.
- ISO/IEC/IEEE 15288:2023. Systems and Software Engineering: System Life Cycle Processes.
- ISO 26262:2018. Road Vehicles: Functional Safety.
- IEC 61508:2010. Functional Safety of E/E/PE Safety-Related Systems.
- RTCA DO-178C. (2011). Software Considerations in Airborne Systems and Equipment Certification.
- RTCA DO-254. (2000). Design Assurance Guidance for Airborne Electronic Hardware.
- SAE ARP4754A. (2010). Guidelines for Development of Civil Aircraft and Systems.
- SAE ARP4761. (1996). Guidelines and Methods for Conducting the Safety Assessment Process on Civil Airborne Systems and Equipment Certification.
- Meyer, B. (1986). Design by Contract. Technical Report TR-EI-12/CO.
- UL 4600. (2020). Standard for Safety for the Evaluation of Autonomous Products.
- ISO/PAS 8800. (2024). Road Vehicles: Safety and Artificial Intelligence.
- Incer, I. (2022). The Algebra of Contracts. Ph.D. thesis, UC Berkeley.
- Tarski, A. (1951). A Decision Method for Elementary Algebra and Real Closed Fields. University of California Press.
- ISO 10007:2017. Quality Management: Guidelines for Configuration Management.
- ISO 9001:2015. Quality Management Systems: Requirements.
- AS9100D:2016. Quality Management Systems: Aviation, Space, and Defense.
- AWS D17.1. (2017). Specification for Fusion Welding for Aerospace Applications.

-----

## 15. License

This specification is published under the MIT License.

Copyright (c) 2026 Substrata Labs LLC

Permission is hereby granted, free of charge, to any person obtaining a copy of this specification, to deal in the specification without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the specification, and to permit persons to whom the specification is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the specification.

-----

## Annex A (Informative): Verification Effort and the Declaration Structure

### A.1 Purpose

The OBSA declaration structure (typed constraints, refinement relationships, operating points (for scalar conditions, the cross-product across parameters, Rule 7.7.5), and compositional decomposition) is designed not only for traceability and conformance checking but also to enable intelligent reduction of verification effort. The structure of the declaration set itself contains information that constrains the verification space, reducing the number of tests, simulations, and analyses needed to achieve evidence-complete conformance (Level 3).

This annex describes, informatively, the properties of the declaration structure that support verification effort reduction. The specific algorithms and computational techniques for exploiting these properties are implementation concerns outside the scope of this specification.

### A.2 Monotonicity Exploitation

Many physical and engineered systems exhibit monotonic relationships between operating condition parameters and constraint metrics. When a constraint metric degrades monotonically with increasing (or decreasing) values of an operating condition parameter, evidence at the worst-case extreme of that parameter implies satisfaction across the full range.

If monotonicity can be established (through analysis evidence, physical reasoning, or limited testing), the verification matrix can be reduced to worst-case corners rather than requiring evidence across the full set of operating points; a designed subset (worst-case corner, pairwise) is declared per obligation with a rationale (Rule 7.7.6). For an n-dimensional operating condition space with k monotonic dimensions, the verification points reduce from O(mⁿ) (where m is the number of sample points per dimension) to O(mⁿ⁻ᵏ), yielding exponential savings in the monotonic dimensions.

Monotonicity is itself an assurance claim and MAY be expressed as an obligation with constraint type CT-BOOLEAN ("the metric is monotonically non-increasing with respect to parameter P") verified by analysis or limited testing.

### A.3 Refinement Margin Concentration

The refinement relationship (Rule 7.3) establishes that design constraints are at least as strict as obligation constraints. The margin between design and obligation constraints indicates where additional evidence has the highest value:

- **Large margin:** The design's validated capability significantly exceeds the obligation's requirement. Evidence production can be less dense in this region because individual evidence failures are unlikely to violate the obligation even if they violate the design constraint.
- **Thin margin:** The design's validated capability barely exceeds the obligation's requirement. Evidence production should be concentrated in this region because the containment decision is sensitive to the actual boundary location.

Implementations MAY use the refinement margin to prioritize evidence production, concentrating test and simulation resources at operating conditions where the margin between design and obligation constraints is smallest.

### A.4 Cross-Obligation Constraint Propagation

When multiple obligations apply to the same component or system, their constraints collectively define the feasible operating region. The intersection of applicable constraints may be tighter than any individual obligation:

- A performance obligation may require response time ≤ T at speed ≥ S.
- A safety obligation may require stopping distance ≤ D at the same speed S.
- The effective constraint on the system at speed S is the intersection of both requirements.

Implementations MAY propagate constraints across obligations to compute the effective constraint surface, which may be a proper subset of any individual obligation's constraint space. Evidence produced at the effective constraint boundaries addresses multiple obligations simultaneously, reducing total verification effort.

### A.5 Composition-Driven Envelope Reduction

When a parent obligation decomposes into children with different design shapes, the child envelopes collectively constrain the parent. The parent's actual achievable performance is bounded by the intersection of its children's validated capabilities. This intersection may be smaller than the parent obligation's required region, meaning the "interesting" verification region (where the parent obligation's satisfaction depends on the evidence) is only at the boundaries of child capability, not across the full operating space.

Implementations MAY use child envelope boundaries to identify the critical operating points for parent-level evidence production, focusing it at conditions where child capabilities limit the parent's performance margin.

### A.6 Combining Evidence Across Environments

Dense evidence can come from inexpensive environments in the regions where they are credible for the quantity measured, anchored at corners and thin-margin regions by environments in which the quantity is directly observed on the real system. The model-validation pattern (Section 4.8.1) is the link between the two. No environment is authoritative by rank: a simulation can be the credible environment for logic, protocol, destructive or high-volume statistical claims where a hardware rig is not, and hardware is the only credible environment for others (contact, thermal).

### A.7 Relationship to Verification Effort

The verification matrix (Rule 7.7.3) defines the total evidence requirement: the demanded conditions C (Rule 9.3.1). Without exploiting the declaration structure, achieving Level 3 conformance requires evidence for every cell in this matrix.

The properties described in this annex reduce the *effective* verification matrix by identifying cells whose evidence is implied by other cells (monotonicity), cells where evidence production has highest value (margin concentration), cells that satisfy multiple obligations simultaneously (constraint propagation), and regions where evidence from inexpensive listed environments is credible (Annex A.6).

The total verification effort is a function of:

- The size of the verification matrix (determined by the declaration set)
- The structure of the constraints (which enables reduction)
- The available verification environments (which determine cost per evidence point)
- The physical characteristics of the system (which determine monotonicity, interaction structure, and margin distribution)

The OBSA structure provides the information needed for an implementation to compute an efficient verification strategy. The strategy itself (which tests to run, which simulations to execute, which analyses to perform) is an implementation concern that depends on domain knowledge, available resources, and risk tolerance.

-----

## Annex B (Informative): The Verification Campaign as a Projection

### B.1 The Campaign as a Computed Projection

Revisions before 0.3.0 make the verification **matrix** computable: which obligation requires what evidence, at which operating points, in which environments (Rule 7.7.3). They stop at enumeration. The act of **producing** that evidence (designing a physical run, deciding what to exercise together, attributing one capture back to many obligations) has remained outside the model, performed by hand and taken on trust.

Revision 0.3.0 closes that loop. It introduces the vocabulary needed to make the **verification campaign** itself a computed projection of the obligation graph, in any environment:

> Given the obligations, their operating conditions, and their measurement footprints, the evaluation engine computes the minimal set of verification runs, the feasible trajectory and instrumentation of each run, and the attribution of each run's capture to individual obligation verdicts. The run plan is derived, not authored; coverage is proven against the matrix, not asserted; per-obligation granularity is preserved by projection.

The mechanism is **environment-invariant**. It is a property of the obligation and its measurement, not of the environment. A verification environment (a CI job, a simulation, a hardware rig, an integrated-system test, the field) is a parameter vector over the same model: how much of the trajectory it controls, how reversibly, at what cost, and how much it can observe. The physical campaign, the simulation suite, the CI matrix, and the field-coverage report become the same projection under different parameters.

### B.2 Core Model

- The system's operating regime is a point in an **operating state space** S, the product of typed **operating dimensions** (velocity, payload, ambient temperature, actuator temperature, state of charge, surface slope, flight mode, and so on).
- An **OperatingCondition** is a **region** in S; a single scalar parameter with a range is the one-dimensional case (Rule 4.4.2).
- A **VerificationRun** is a **trajectory** through S over time. A VP's evidence is valid only while the trajectory sits in that VP's required region, over the VP's measurement window.
- Compatibility, exclusivity, ordering, and interference between verifications are **computed** from region geometry plus each environment's effective controllability of the dimensions involved (Rule 4.5.1), in the same status as the computed fields of Rule 7.11. They are never authored pairwise.

-----

## Annex C (Normative for Engines): Migration and Deprecations

### C.1 Purpose

This annex lists, for each core revision, the deprecated forms an evaluation engine MUST accept and the rules whose violations an engine MUST report as warnings rather than violations during their introduction (Rule 11.3.4). A deprecated form is accepted, with a warning, for one minor version. Changes to the fixed core are recorded per Rule 11.3.3.

### C.2 Revision 0.2.2

|Form or rule|Treatment|
|------------|---------|
|Obligation `min_fidelity` (Section 4.2) and VerificationEnvironment `fidelity` (Section 4.5)|Deprecated. Accepted with a warning; carry no semantics and are ignored by evaluation. Removed in 0.3.0.|
|`source_type: safety_analysis` (Section 4.1)|Deprecated alias of `hazard_analysis`. Accepted with a warning and evaluated as `hazard_analysis`.|
|VerificationProcedure `provenance` (Section 4.7.1)|Deprecated alias of `derivation`. Accepted with a warning and evaluated as `derivation`.|
|A single `operating_condition` ID on a verification procedure or evidence entry (Sections 4.7.1, 4.7.2)|Accepted; denotes a one-condition operating point (Rule 7.7.5). Where the obligation's applicable conditions constrain more than one parameter, such an entry matches none of its operating points and discharges no condition (`no-evidence`, Rule 9.3.4); the engine reports each such entry with a warning (Rule 11.3.4). A declaration set that was evidence-complete under 0.2.1 with single-condition evidence may not be evidence-complete under this revision.|
|Obligation `elements` (Section 4.2)|SHOULD. Engines MAY suggest `elements` from the components of satisfying designs for human confirmation, and MUST NOT default it silently.|
|Rule 7.1.5 (source-artifact clause), Rule 7.1.6, Rule 7.2.4, Rule 7.4.1 (unit-consistency clause), Rule 7.7.7, Section 4.8.1 (an analysis that is also a requirement source as sole evidence)|Introduced in 0.2.2. Violations are reported as warnings.|
|Fidelity hierarchy|Removed from the fixed core (Section 11.1, Rule 11.3.3).|

### C.3 Revision 0.3.0

|Form or rule|Treatment|
|------------|---------|
|Obligation `min_fidelity` and VerificationEnvironment `fidelity`|Removed from the schema, after deprecation in 0.2.2.|
|OperatingCondition `parameter` and `range` (Section 4.4)|Accepted as an alias of `region` (Rule 4.4.2). A dimension named only by scalar-form conditions and not declared is implicitly declared, with a warning (Rule 4.4.1).|
|Obligation `elements` (Section 4.2)|MUST. Its count and design inclusion are checked by Rule 7.2.4.|
|Rule 7.1.5 (source-artifact clause), Rule 7.1.6, Rule 7.2.4, Rule 7.4.1 (unit-consistency clause), Rule 7.7.7, Section 4.8.1 (an analysis that is also a requirement source as sole evidence)|No longer warning-only: violations are reported as violations.|
|VerificationProcedure `provenance` (Section 4.7.1)|Remains a deprecated alias of `derivation` through 0.3.x, accepted with a warning.|
|Requirement `record_id` (Section 4.1)|Remains optional. Absence is reported as an Unrecorded Requirement gap (Section 9.1), not a structural violation.|

-----

## Annex D (Informative): Profiles

A profile is a deployment-specific restriction of this specification: for example, the admitted source kinds and the requirements system of record whose identifiers fill `record_id`. Example: a deployment profile admits `hazard_analysis`, `fmea`, `capa`, `product_requirement` and `interface_contract`, names the organization's requirements management tool as the system of record, and pins the export format version of that tool used for coverage queries (Section 1.6). A profile MAY also make the Unrecorded Requirement gap blocking for gated execution.

-----

## Revision History

| Version | Date | Notes |
|---------|------|-------|
| 0.1.0-draft | 2026-02-18 | Working draft. Added CT-CONTAINMENT-ND constraint type for multi-dimensional envelope support. Added multi-dimensional fields to Envelope design shape (Section 5.3). Added Rule 7.10 for state-dependent obligation activation via mode_obligation_map. Added Rule 9.2.4 for runtime evidence ingestion. Added containment composition rule to Rule 7.4.1. Added model validation informative note to Section 4.7.1. Added Annex A on verification effort reduction. |
| 0.2.0-draft | 2026-03-03 | Field classification and explicit binding. Replaced `min_verification_fidelity` with `min_fidelity`. Added `applicable_conditions` and `applicable_environments` as explicit obligation fields. Removed `children`, `satisfied_by`, `decomposed_into` from authored schema (now computed, Rule 7.11). Added Component instance type (Section 4.6). Added VP `maps_to` field and deduplication rule (Rule 7.9.4). Removed `interface_id` from Design; Interface is now a computed view (Section 4.7.7). Added structured Flow schema for Exchange designs (Section 5.1). Added expression grammar and `decidable` field for CT-CONTAINMENT-ND. Added Structural Parametricity (Section 3.1). Added Composition Across Declaration Sets (Section 8.5). Added Evaluation Engine Requirements (Section 1.6). Added VP `provenance` field and authored procedure governance. Added Composition gap type. Added Rule 10.2.4 fidelity consistency check. Added Section 10.3 transitional field documentation. Added Section 1.4 single-configuration scope statement. Added informative notes: VE structured capabilities evolution (Section 4.5), applicable_conditions permanence rationale, design VP partial derivability, parent evidence gap clarification (Rule 8.1.1), Exchange Scenario stub marker. |
| 0.2.1-draft | 2026-03-07 | Evidence traceability and staleness. Added `component_revisions` field to Evidence schema (Section 4.7.2): evidence records which component revisions it was produced against. Added Evidence Stale gap type (Section 9.1): evidence with non-current component revisions. Added Rule 9.2.6 for staleness gap computation. Added Evidence freshness metric to Section 9.3. Expanded conformance from two to three levels: Level 1 (Structurally Conformant), Level 2 (Evidence-Evaluated: evidence exists and has been mechanically evaluated), Level 3 (Evidence-Complete: full coverage, no gaps). Updated all Level 2 references to Level 3 where they referred to evidence-completeness. |
| 0.2.2-draft | 2026-11-19 | Assurance-model revision. Removed the fidelity ordering: Fidelity term, ordering table, Section 10.1 hierarchy, Rules 7.6.3, 10.1.2, 10.2.1, 10.2.2 and 10.2.4 withdrawn; Rule 10.2.3 consolidated into 7.7.3 and 9.3.1; Rule 7.7.4 reserved; `min_fidelity` and VE `fidelity` deprecated; 8.3 restated; fidelity hierarchy removed from the fixed core. Added interim admission by listing (10.2.5) with disclosure (10.2.6), and Rule 7.6.5 with the descriptive VE field `realization`. Demand-side coverage partition, verdict authority, oracle independence, coverage queries, lexical metric identity (9.3.1–9.3.7); operating points (7.7.5–7.7.6); plan-entry environment bindings (7.7.7). Scope by arity and attribution with `elements` (7.2.1–7.2.4). Requirement source kinds, optional `record_id` with the Unrecorded Requirement gap, Rules 7.1.5–7.1.7. New MUSTs introduced as warnings (Rule 11.3.4). `provenance` renamed `derivation`. Tool-qualification disclosure (12.2.5). Core revision and deprecation rules (11.3.3–11.3.4). Section 1.4 clarified. Single gap-type table (9.1). Annexes C and D. VP `provenance` in the 0.2.0 row is now `derivation`. |
| 0.3.0-draft | 2026-11-19 | Operating state space, verification runs, and co-execution. Revised OperatingCondition to be region-valued (Section 4.4) using the Section 5.3 grammar. Added OperatingDimension instance type with exogenous/endogenous/regime kinds and endogenous dynamics (Section 4.4a). Revised the VerificationEnvironment definition from a passive "context in which evidence is produced" to a control + observation authority over the operating state space (a steerable, instrumented harness abstraction; `field` = zero-control degenerate case), updating the Section 2 term and Section 4.5. Added optional VerificationEnvironment structured capabilities (controllable_dimensions with a `mode` of instantaneous or rate_limited, observable_metrics, and cost), filling the Section 4.5 stub and enabling capability validation (10.3.1). Extended feasibility (Rule 7.12.8) from containment (can the VE hold each region) to reachability (can the VE steer the trajectory through the ordered windows within its control authority), with rate_limited between-window traversal as a real covering cost (Rule 7.12.9). Added VerificationRun derived type (Section 4.7.8) with a trajectory sub-schema (Section 4.7.8.1); trajectories are distinct from the Exchange scenario stub (Section 5.1), which remains a stub. Amended Evidence with a `run` reference and the projection note (Section 4.7.2). Added the measurement footprint (stimulus, drives, sensor_channels) to VerificationPlanEntry (Section 4.8.2). Added `terminal` to Mode State (Section 5.4). Added statistical-independence-under-co-execution rule (Rule 6.2.1). Added run OC-set compatibility (Rule 7.7.4). Added Section 7.12 Co-execution and Interference: computed relations (compatible, exclusive, precedes, partitions, stimulus/observability conflict), the conservative default, feasibility, and run covering. Added Co-execution Conflict, Batch Coverage, and Interference Discovery gap types and the divergence rule (Rule 9.2.7). Added environment capability (Rule 10.3.1). Recorded governance: this is a domain-invariant core revision. Two modeling choices resolved, 2026-07-07: dynamics as annotation; per-environment controllability override (Rule 4.5.1). Module drafted 2026-07-07; merged into the core specification 2026-11-19. Scalar OC alias (4.4.2); divergence rule restated across environments (9.2.7); §7.12 restated over effective controllability (4.5.1); capability failure ranked below `failed`; deprecated fidelity fields removed; `elements` required; 0.2.2 warning-only rules become violations. Annex B (the verification campaign as a projection) and Annex C.3. |

Erratum: the 0.1.0 row's 'Section 4.7.1' should read 'Section 4.8.1'.

-----

## Citation

```
Substrata Labs LLC. (2026). OBSA: Obligation-Based Systems Assurance
(Core Specification, Version 0.3.0-draft).
https://github.com/substrata-labs/obsa
```
