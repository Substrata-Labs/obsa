# OBSA: Obligation-Based Systems Assurance

## Minimum Viable Specification

**Version 0.1.0-draft**
**Status: Working Draft**
**Substrata Labs LLC**
**February 2026**

-----

## 1. Introduction and Scope

### 1.1 Purpose

This document specifies OBSA (Obligation-Based Systems Assurance), an ontology for compositional quality assurance in complex systems. OBSA defines a typed declaration hierarchy, structural invariants, composition rules, and conformance criteria that enable automated derivation of verification artifacts, compositional evidence accumulation, and computed traceability.

### 1.2 Novel Contribution

OBSA formalizes the *obligation* as an independent computational artifact occupying the layer between stakeholder requirements and engineering designs. The obligation is both design-independent (it does not prescribe implementation) and directly testable (it can be verified without knowing which design satisfies it). This dual property makes the obligation the correct pivot for automated derivation. The individual concepts (requirements decomposition, contract-based design, verification hierarchies) are established in systems engineering literature. Their organization into a typed, computable hierarchy with formal refinement, compositional evidence, and computed gap analysis is new.

### 1.3 Domain Neutrality

OBSA applies to any domain where components are composed through interfaces and quality depends on constraints that cross boundaries: aerospace vehicles, electronic subsystems, embedded firmware, software services, manufacturing processes, field operations, cloud infrastructure, and fleet management. The declaration hierarchy, structural rules, composition rules, and conformance criteria are domain-invariant. Domain-specific concerns are addressed through harnesses and adapters, which are implementation details outside the scope of this specification.

### 1.4 What This Specification Does Not Define

This specification defines the abstract data model and the rules governing it. It does not define:

- Computational mechanisms, evaluation engines, or algorithms
- File formats, serialization schemes, or wire protocols
- Tooling, user interfaces, or development environments
- Organizational processes, approval workflows, or meeting structures
- Harness or adapter implementations

The specification is the shared language. The computational mechanism is a separate concern.

-----

## 2. Terms and Definitions

Terms are listed alphabetically. Where a term is formally defined as an instance type (Section 4), the definition here is a summary; the normative definition is the instance type schema.

**Acceptance function.** A function defined by a constraint type that evaluates an evidence payload and returns a verdict. The acceptance function determines whether measured, observed, or assessed values satisfy the constraint's assertion.

**Adapter.** A template that renders abstract verification artifacts into domain-specific executables, procedures, or inspection criteria. One adapter per target tool or environment.

**Budget (design shape).** A design shape modeling a finite resource with supply, demand, allocation, and capacity constraints.

**Compatibility (design shape).** A design shape modeling agreement on versions, parameters, tolerances, or specifications across a boundary between components.

**Component.** A discrete element that performs work or contributes a property. The atomic unit of system decomposition. Has intrinsic properties independent of its connections.

**Composition.** The process by which independently verified child obligations combine to satisfy a parent obligation.

**Conformance.** The property of a declaration set satisfying the structural rules and, optionally, the evidence completeness criteria defined in this specification.

**Constraint.** A specific, parameterized assertion that an obligation requires to hold. Typed by one of the recognized constraint types.

**Constraint type.** A category of assertion pattern with a defined refinement rule and acceptance function. See Section 6.

**Decomposition gap.** A gap where a requirement exists but has not been decomposed into obligations.

**Design.** The physical realization of one or more obligations. Binds obligations to specific components, materials, protocols, parameters, and constraints. Instantiated as one of the recognized design shapes. The locus of evidence production.

**Design shape.** A category of design determined by the kind of relationship it models. See Section 5.

**Envelope (design shape).** A design shape modeling the boundary of validated capability, operating range, or characterization evidence across one or more dimensions.

**Evidence.** An artifact produced by verification execution, inspection, analysis, or demonstration that contains a structured measurement payload evaluable by a constraint type's acceptance function. Evidence is produced at and attached to the design level.

**Exchange (design shape).** A design shape modeling structured transfer of data, energy, material, or force between components.

**Fidelity.** The degree to which a verification environment approximates the production system. Ordered: bench < SIL < HIL < system_test < field.

**Gap.** A structural deficiency in the declaration set where an obligation lacks required evidence. Classified into four types: decomposition, scenario, infrastructure, observability.

**Harness.** A mapping from constraint types to abstract verification artifacts for a specific interface medium or verification domain.

**Infrastructure gap.** A gap where an obligation exists but no harness can derive verification artifacts for its medium.

**Interaction constraint.** A constraint on the joint relationship between two or more dimensions of an n-dimensional envelope, expressing a coupling that cannot be captured by independent per-dimension ranges alone.

**Interface.** A coupling between two components through which energy, material, data, or force is exchanged.

**Mode (design shape).** A design shape modeling valid states, transitions, guards, and invariants for a stateful element.

**Monotonic gap closure.** The property that the set of active obligations only grows (never shrinks) and the evidence base only grows, so the gap between validated coverage and encountered conditions decreases monotonically.

**Obligation.** A testable, design-independent constraint allocated to a scope and level. The pivot of the OBSA architecture. See Section 4.2.

**Observability gap.** A gap where an obligation exists but no telemetry mapping can produce runtime measurements for it.

**Operating condition.** A declared environment parameter that the system must tolerate.

**Refinement.** The relationship between a design and an obligation where the design's constraints are at least as strict as the obligation's constraints. Refinement establishes that evidence satisfying the design's constraints necessarily satisfies the obligation's constraints.

**Requirement.** A statement of need from an external source. The top of the declaration hierarchy.

**Scenario gap.** A gap where a field or production observation has no corresponding obligation.

**System.** The composition of all components and interfaces. Has emergent properties not attributable to any single component or interface.

**Traceability.** The computed property of the declaration dependency graph (DAG) that every requirement can be traced through obligations and designs to evidence. Not maintained; derived.

**Verdict.** A computed assessment of whether an obligation is satisfied, derived from evidence attached to its satisfying designs and evaluated through the refinement relationship.

**Verification environment.** The physical or simulated context in which evidence is produced. Characterized by fidelity level.

**Verification method.** One of the four recognized methods for producing evidence: test, inspection, analysis, demonstration (TIAD). See Section 4.7.

**Verification plan.** A structured mapping from constraint parameters to verification methods, declared on both obligations and designs. Enables mechanical matching of evidence coverage.

-----

## 3. Architecture Overview

OBSA organizes declarations into three layers connected by typed references:

```
┌─────────────────────────────────────────────────────┐
│  REQUIREMENTS LAYER                                 │
│  Stakeholder needs, safety analyses, regulations    │
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

Traceability is a latent property of the dependency graph. Gap analysis is a computed comparison of obligations against evidence.

-----

## 4. Instance Types

### 4.1 Requirement

A statement of need from an external source.

|Field           |Type              |Required|Description                                                                                                                       |
|----------------|------------------|--------|----------------------------------------------------------------------------------------------------------------------------------|
|id              |string            |yes     |Unique identifier. Format: `REQ-{domain}-{seq}` recommended.                                                                      |
|title           |string            |yes     |Human-readable name.                                                                                                              |
|source_type     |enum              |yes     |One of: `safety_analysis`, `regulatory`, `customer`, `conops`, `manufacturing_standard`, `field_reliability`, `internal_standard`.|
|source_reference|string            |yes     |Specific clause, section, analysis ID, or document reference.                                                                     |
|text            |string            |yes     |The requirement statement. Should use "shall" language.                                                                           |
|rationale       |string            |no      |Why this requirement exists.                                                                                                      |
|decomposed_into |list<ObligationID>|no      |Obligation IDs this requirement decomposes into. Empty list indicates a decomposition gap.                                        |

**Cardinality:** Many. Hundreds in a mature system.

**Example:**

```yaml
id: REQ-SAFE-001
title: Single Engine Continued Takeoff
source_type: safety_analysis
source_reference: FHA-2025-003, Hazard H7; 14 CFR 25.121(a)
text: "The aircraft shall maintain a positive climb gradient following
       a single engine failure recognized at or above V1 during takeoff."
decomposed_into: [OBL-SYS-OEI-CLIMB]
```

### 4.2 Obligation

A testable, design-independent constraint allocated to a scope and level.

|Field                    |Type                         |Required|Description                                                                                                                    |
|-------------------------|-----------------------------|--------|-------------------------------------------------------------------------------------------------------------------------------|
|id                       |string                       |yes     |Unique identifier. Format: `OBL-{level}-{name}` recommended.                                                                   |
|title                    |string                       |yes     |Human-readable name.                                                                                                           |
|parent                   |RequirementID or ObligationID|yes     |What this obligation was derived from.                                                                                         |
|source_type              |enum                         |yes     |One of: `requirement_decomposition`, `engineering_analysis`, `field_discovery`, `manufacturing_nonconformance`, `test_failure`.|
|scope                    |enum                         |yes     |One of: `intrinsic` (single component), `relational` (interface between components), `emergent` (system-level).                |
|level                    |enum                         |yes     |One of: `system`, `subsystem`, `component`.                                                                                    |
|status                   |enum                         |yes     |One of: `active`, `superseded`, `withdrawn`. Default: `active`. See Rule 8.4.                                                  |
|constraint_type          |ConstraintTypeID             |yes     |Which constraint type (Section 6).                                                                                             |
|constraint               |Constraint                   |yes     |The specific parameterized assertion. See Section 6 for structure per type.                                                    |
|verification_plan        |list<VerificationPlanEntry>  |yes     |Mapping of constraint parameters to verification methods. At least one entry required. See Section 4.7.                        |
|applicable_conditions    |list<OperatingConditionID>   |no      |Defaults to ALL conditions if omitted.                                                                                         |
|condition_exemptions     |list<OperatingConditionID>   |no      |Conditions explicitly exempted. Safety obligations MUST NOT have exemptions (Rule 7.6).                                        |
|min_verification_fidelity|enum                         |no      |One of: `bench`, `SIL`, `HIL`, `system_test`, `field`. If omitted, defaults to `bench`.                                        |
|children                 |list<ObligationID>           |no      |Child obligations in decomposition hierarchy.                                                                                  |
|satisfied_by             |list<DesignID>               |no      |Design(s) satisfying this obligation. Empty before design exists.                                                              |

**Cardinality:** Dozens to hundreds. System-level obligations are few (order 10). Subsystem obligations are moderate (order 10 per subsystem boundary). Component obligations are numerous (order 10 per component).

**Scope definitions:**

|Scope     |Applies to                              |Verified by                            |Example                                                                            |
|----------|----------------------------------------|---------------------------------------|-----------------------------------------------------------------------------------|
|intrinsic |Single component's property             |Bench test, unit test, inspection      |Engine thrust rating, weld joint shear strength, API response time                 |
|relational|Property of interface between components|Integration test, assembly verification|ARINC 429 bus timing, bolt pattern compatibility, API schema match                 |
|emergent  |Property of composed system             |System test, field observation         |Continued takeoff climb gradient, fleet dispatch reliability, total aircraft weight|

**Example:**

```yaml
id: OBL-SYS-OEI-CLIMB
title: One-Engine-Inoperative Climb Gradient
parent: REQ-SAFE-001
source_type: requirement_decomposition
scope: emergent
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
  - parameter: aerodynamic_model_correlation
    method: analysis
    rationale: "CFD model validated against wind tunnel and flight test data"
  - parameter: pilot_engine_failure_recognition
    method: demonstration
    rationale: "Witnessed simulator evaluation per AC 25-7D"
min_verification_fidelity: system_test
children:
  - OBL-SUB-ENGINE-THRUST
  - OBL-SUB-DRAG-BUDGET
  - OBL-SUB-FCS-ASYMMETRY
  - OBL-SUB-FADEC-RESPONSE
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
|components       |list<ComponentID>          |yes        |At least one. Components involved.                                                                                                                           |
|interface_id     |InterfaceID                |conditional|Required if design involves a relational obligation.                                                                                                         |
|constraints      |list<Constraint>           |yes        |Design-level constraints. Must refine obligation constraints per Rule 7.3.                                                                                   |
|verification_plan|list<VerificationPlanEntry>|yes        |Mapping of design parameters to verification methods. Must cover all obligation verification plan parameters for obligations in `satisfies`. See Section 4.7.|
|version          |string                     |no         |Version of this design declaration.                                                                                                                          |

**Cardinality:** Varies by shape. See Section 5.

### 4.4 OperatingCondition

A declared environment parameter the system must tolerate.

|Field    |Type  |Required|Description                                                                                                          |
|---------|------|--------|---------------------------------------------------------------------------------------------------------------------|
|id       |string|yes     |Unique identifier.                                                                                                   |
|title    |string|yes     |Human-readable name.                                                                                                 |
|parameter|string|yes     |Environmental factor: temperature, altitude, vibration, humidity, icing, wind_shear, turbulence, supply_voltage, etc.|
|range    |Range |yes     |The range of values to tolerate. Structure: `{min, max, unit}`.                                                      |
|severity |enum  |no      |One of: `nominal`, `degraded`, `extreme`.                                                                            |

**Cardinality:** Typically 10-50 per system.

**Example:**

```yaml
id: OC-ALTITUDE-TAKEOFF
title: Takeoff Field Elevation Range
parameter: pressure_altitude
range: {min: 0, max: 8000, unit: ft}
severity: nominal
```

### 4.5 VerificationEnvironment

The physical or simulated context in which evidence is produced.

|Field       |Type        |Required|Description                                           |
|------------|------------|--------|------------------------------------------------------|
|id          |string      |yes     |Unique identifier.                                    |
|fidelity    |enum        |yes     |One of: `bench`, `SIL`, `HIL`, `system_test`, `field`.|
|title       |string      |yes     |Human-readable name.                                  |
|description |string      |yes     |What physical or simulated context this represents.   |
|capabilities|list<string>|no      |What can be verified at this fidelity.                |

**Fidelity ordering (lowest to highest):**

|Fidelity   |Approximation                             |Examples                                                               |
|-----------|------------------------------------------|-----------------------------------------------------------------------|
|bench      |Component in isolation                    |Dynamometer, load cell, signal generator, unit test runner             |
|SIL        |All elements simulated                    |Flight dynamics model, simulated avionics, CI test environment         |
|HIL        |Real hardware, simulated counterpart      |Real FADEC on test rig, simulated engine plant model, real ARINC bus   |
|system_test|Full system, controlled conditions        |Iron bird rig, complete avionics integration bench, staging environment|
|field      |Production system, uncontrolled conditions|Flight test aircraft, production fleet, live ground infrastructure     |

### 4.6 Derived Instance Types

These types are produced by computation, not authored by humans. Their schemas are informative (implementations may vary in structure while preserving semantics).

#### 4.6.1 VerificationProcedure

|Field                   |Type                     |Description                                                                                 |
|------------------------|-------------------------|--------------------------------------------------------------------------------------------|
|id                      |string                   |Unique identifier.                                                                          |
|derived_from_design     |DesignID                 |The design this procedure verifies.                                                         |
|obligation              |ObligationID             |The obligation this procedure ultimately addresses (via the design's `satisfies` reference).|
|parameter               |string                   |The specific constraint parameter under verification. From the design's verification plan.  |
|method                  |enum                     |`test`, `inspection`, `analysis`, `demonstration`. From the verification plan entry.        |
|operating_condition     |OperatingConditionID     |The condition under which the procedure executes.                                           |
|verification_environment|VerificationEnvironmentID|The fidelity at which the procedure runs.                                                   |
|constraint_under_test   |Constraint               |The design-level constraint being checked.                                                  |
|procedure               |domain-specific          |The executable test, inspection checklist, analysis model, or demonstration protocol.       |

#### 4.6.2 Evidence

Evidence is produced at the design level. Obligation satisfaction is computed from design-level evidence through the refinement relationship.

|Field                   |Type                     |Description                                                                                |
|------------------------|-------------------------|-------------------------------------------------------------------------------------------|
|id                      |string                   |Unique identifier.                                                                         |
|verification_procedure  |VerificationProcedureID  |Which procedure produced this evidence.                                                    |
|design                  |DesignID                 |Which design this evidence is attached to.                                                 |
|parameter               |string                   |Which constraint parameter was measured, observed, or assessed.                            |
|method                  |enum                     |`test`, `inspection`, `analysis`, `demonstration`.                                         |
|operating_condition     |OperatingConditionID     |Under which condition.                                                                     |
|verification_environment|VerificationEnvironmentID|At which fidelity.                                                                         |
|measurement             |Measurement              |Structured payload evaluable by the constraint type's acceptance function. See Section 6.2.|
|result                  |enum                     |`pass`, `fail`, `inconclusive`. Computed by the acceptance function.                       |
|timestamp               |datetime                 |When evidence was produced.                                                                |
|artifact_reference      |string                   |Link to raw test result, inspection record, analysis report, demonstration log.            |

#### 4.6.3 Verdict

|Field        |Type        |Description                                                                                                                                  |
|-------------|------------|---------------------------------------------------------------------------------------------------------------------------------------------|
|obligation   |ObligationID|Which obligation.                                                                                                                            |
|status       |enum        |`satisfied` (all conditions pass via design evidence), `violated` (any condition fails), `incomplete` (missing evidence for some conditions).|
|coverage     |float       |Fraction of (parameter × method × condition) tuples with passing evidence across all satisfying designs.                                     |
|blocking_gaps|list<Gap>   |Specific gaps preventing full satisfaction.                                                                                                  |

Verdict computation: For an obligation O, find all designs D where O ∈ D.satisfies. For each D, check that evidence exists covering all parameters in O's verification plan, under all applicable conditions, at fidelity ≥ O.min_verification_fidelity. For each piece of evidence, apply the constraint type's acceptance function to the measurement payload against D's constraint. If D's constraint is satisfied, then by refinement (Rule 7.3), O's constraint is necessarily satisfied.

#### 4.6.4 Gap

|Field      |Type  |Description                                                                        |
|-----------|------|-----------------------------------------------------------------------------------|
|type       |enum  |`decomposition`, `scenario`, `infrastructure`, `observability`.                    |
|subject    |ID    |The requirement, obligation, or observation involved.                              |
|description|string|Human-readable description of the gap.                                             |
|nature     |enum  |`knowledge` (decomposition, scenario) or `tooling` (infrastructure, observability).|

#### 4.6.5 RuntimeConstraint

|Field       |Type        |Description                                 |
|------------|------------|--------------------------------------------|
|id          |string      |Unique identifier.                          |
|derived_from|ObligationID|Source obligation.                          |
|check       |string      |The runtime assertion expression.           |
|on_violation|enum        |`safe_shutdown`, `degrade`, `log_and_alert`.|
|severity    |enum        |`safety`, `operational`, `monitoring`.      |

#### 4.6.6 TraceabilityChain

Not a stored artifact. The dependency DAG itself is the traceability. For any node in the graph, the chain is computed by traversal: Requirement → Obligation → Design → VerificationProcedure → Evidence → Verdict. Every link is traversable in both directions.

### 4.7 Verification Methods and Verification Plans

#### 4.7.1 Verification Methods (TIAD)

OBSA recognizes four verification methods, consistent with the TIAD taxonomy established in ARP4754A, DO-178C, and INCOSE systems engineering practice:

|Method       |Description                                                   |Evidence Produced By                                                 |Examples                                                                    |
|-------------|--------------------------------------------------------------|---------------------------------------------------------------------|----------------------------------------------------------------------------|
|test         |Automated or manual execution that produces measurable results|Execution of a procedure against the system or component             |CI pact tests, integration tests, unit tests, dynamometer runs, flight tests|
|inspection   |Review of artifacts against criteria (checklist-based)        |Examination of physical or digital artifacts against defined criteria|Code review, wiring review, document review, dimensional inspection         |
|analysis     |Modeling, simulation, or expert assessment against a standard |Computational modeling, simulation, or domain expert evaluation      |CFD analysis, worst-case execution time analysis, stress analysis, FMEA     |
|demonstration|Observed execution showing capability works                   |Witnessed operation under defined conditions                         |Simulator evaluation, field demonstration, witnessed walkthrough            |

The choice of verification method for a given parameter is authored by domain experts, not derived algorithmically. It reflects engineering judgment, applicable standards, and the nature of the constraint being verified.

**Informative note: model validation for analysis-derived evidence.**

When evidence is produced by the `analysis` method using a computational model (simulation, surrogate model, trained predictor), the validity of the model is itself an assurance concern. Model validation MAY be expressed as a separate obligation with constraints on model accuracy (e.g., prediction error thresholds, correlation coefficients against physical measurements) and evidence from comparison of model outputs against higher-fidelity results. This pattern ensures that analysis-derived evidence is traceable to validated modeling assumptions.

The model validation obligation follows standard OBSA structure: it has a parent (the obligation whose analysis evidence depends on the model), a constraint type (typically CT-THRESHOLD on prediction error or CT-STATISTICAL on correlation), a verification plan (typically `test`, comparing model output to physical measurement), and evidence attached to a design. No special-purpose types are required. The general pattern is:

1. A primary obligation O requires evidence via `analysis` method.
2. A model validation obligation M is declared with O as parent (or the same parent requirement as O).
3. M's constraint specifies acceptable model accuracy (e.g., prediction error ≤ 5%).
4. M's evidence comes from `test` method: comparison of model predictions against physical measurements at higher fidelity.
5. O's analysis evidence is trustworthy because M is satisfied.

This pattern applies to any domain where computational models are used as evidence sources: aerodynamic simulations, structural analysis, process capability predictions, throughput models, and environmental simulations.

#### 4.7.2 Verification Plan

A verification plan is a list of verification plan entries declared on both obligations and designs. Each entry maps a specific constraint parameter to a verification method.

**VerificationPlanEntry:**

|Field    |Type  |Required|Description                                                                                                             |
|---------|------|--------|------------------------------------------------------------------------------------------------------------------------|
|parameter|string|yes     |A constraint metric or design shape field. References a named value already declared in the constraint or design schema.|
|method   |enum  |yes     |One of: `test`, `inspection`, `analysis`, `demonstration`.                                                              |
|rationale|string|no      |Why this method was selected for this parameter.                                                                        |

**Obligation verification plan:** Declares what parameters require evidence and by which methods. Parameters reference the obligation's constraint metrics or, for decomposed obligations, the parameters of the contributing physical quantities.

**Design verification plan:** Declares how the design's parameters will be verified. Must cover all parameters declared in the verification plans of all obligations in the design's `satisfies` list. The design's verification plan may include additional parameters specific to the design's shape and medium.

**Mechanical matching rule:** For each entry in an obligation's verification plan, at least one entry in the satisfying design's verification plan must reference the same parameter (or a design-level parameter that maps to the obligation parameter through the constraint) with the same verification method. This matching is checked as part of Level 1 conformance (Section 12).

**Example: obligation verification plan.**

```yaml
# On OBL-SUB-FCS-ASYMMETRY-COMP
verification_plan:
  - parameter: asymmetry_compensation_time
    method: test
    rationale: "Measured response time on HIL rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case scheduling analysis"
```

**Example: design verification plan.**

```yaml
# On DES-EXCH-FCS-FADEC
verification_plan:
  - parameter: asymmetry_compensation_time
    method: test
    rationale: "ARINC 429 bus timing measurement on HIL rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case execution time analysis of FCS software"
  - parameter: thrust_asymmetry_command_rate
    method: inspection
    rationale: "ICD review confirms 50Hz label 312 configuration"
```

### 4.8 TelemetryMapping (Stub)

A telemetry mapping declares that a runtime measurement source exists for an obligation's metric, enabling observability gap detection.

|Field     |Type        |Required|Description                                                                           |
|----------|------------|--------|--------------------------------------------------------------------------------------|
|id        |string      |yes     |Unique identifier.                                                                    |
|obligation|ObligationID|yes     |The obligation whose metric is observable.                                            |
|metric    |string      |yes     |The constraint metric being observed. Must match the obligation's constraint metric.  |
|source    |string      |yes     |The telemetry source: sensor ID, log stream, monitoring endpoint, data bus label, etc.|

**Cardinality:** One per observable obligation metric. Absence of a TelemetryMapping for an obligation's metric produces an observability gap (Section 9).

The internal structure and runtime behavior of telemetry mappings is an implementation concern outside this specification. This type exists solely to enable mechanical detection of observability gaps.

-----

## 5. Design Shapes

Designs are instantiated as one of the recognized shapes. Each shape models a distinct kind of relationship with distinct constraint types and distinct validation logic.

### 5.1 Exchange

Models structured transfer of data, energy, material, or force between components.

|Field    |Type          |Required|Description                                                                                                                       |
|---------|--------------|--------|----------------------------------------------------------------------------------------------------------------------------------|
|medium   |string        |yes     |The coupling medium: ARINC_429, ARINC_664, CAN, MIL-STD-1553, gRPC, bolted_joint, thermal_path, fluid_coupling, optical_link, etc.|
|flows    |list<Flow>    |yes     |Typed items exchanged. Each flow has: name, direction (in/out/bidirectional), type/format, timing constraints.                    |
|scenarios|list<Scenario>|no      |Behavioral sequences describing interaction patterns (request-response, cyclic, event-driven, failure annunciation).              |

**Domain examples:**

- FADEC ↔ flight computer over ARINC 429: thrust command words, engine parameter reports, failure annunciations
- Turbine disk ↔ shaft spline joint: torque transmission, axial load path, thermal expansion coupling
- Hydraulic pump ↔ flight control actuator: pressure, flow rate, response time
- Ground operations server ↔ airline dispatch over ACARS: flight plan uplinks, weather updates, acknowledgments
- Assembly station input ↔ output: part flow, cycle time, quality handoff criteria

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
|interaction_constraints  |list<InteractionConstraint>      |no         |Constraints on the joint relationship between two or more dimensions. Each has: parameters (list<string>), expression (string), rationale (string).     |
|representation           |enum                             |no         |One of: `hyperrectangle`, `hyperrectangle_with_interactions`, `convex_hull`, `parametric_boundary`. Defaults to `hyperrectangle` if no interactions declared, `hyperrectangle_with_interactions` if interactions are present.|
|characterization_method  |string                           |no         |How the envelope was established (test, analysis, simulation, field data, or combination).                                                              |
|characterization_evidence|list<EvidenceID>                 |no         |References to evidence establishing the envelope. Multiple references support envelopes characterized by combining evidence at different fidelities.    |
|characterization_points  |integer                          |no         |Number of characterization points used to establish the envelope boundary.                                                                              |

An Envelope design uses EITHER the scalar fields (for a single dimension) OR the multi-dimensional fields (for two or more dimensions), not both. The scalar form is a convenience for the common single-dimension case and is semantically equivalent to a multi-dimensional envelope with one dimension and no interaction constraints.

**EnvelopeDimension:**

|Field |Type  |Required|Description                                  |
|------|------|--------|---------------------------------------------|
|name  |string|yes     |The parameter being bounded.                 |
|range |Range |yes     |The validated range: `{min, max, unit}`.     |

**InteractionConstraint:**

|Field     |Type        |Required|Description                                                                                                    |
|----------|------------|--------|---------------------------------------------------------------------------------------------------------------|
|parameters|list<string>|yes     |Names of the interacting dimensions. Must reference names declared in the `dimensions` list. At least two.     |
|expression|string      |yes     |A constraint expression over the named parameters. Must be evaluable as a boolean.                             |
|rationale |string      |no      |Why this interaction exists (physical coupling, empirical observation, safety analysis).                        |

**Representation semantics:**

- `hyperrectangle`: The validated region is the Cartesian product of the per-dimension ranges. Each dimension is independent.
- `hyperrectangle_with_interactions`: The validated region is the Cartesian product of per-dimension ranges intersected with the interaction constraints. Interaction constraints carve out portions of the hyperrectangle where dimensions are jointly constrained.
- `convex_hull`: The validated region is the convex hull of characterization points. Used when the envelope shape is empirically determined and does not decompose cleanly into axis-aligned ranges. The characterization points are referenced through `characterization_evidence`.
- `parametric_boundary`: The validated region is defined by a level set of a function (e.g., `f(x) ≥ threshold`). The function is a validated model referenced through `characterization_evidence`. Model validation follows the pattern described in Section 4.7.1.

**Domain examples:**

- Engine thrust curve: sea level to 45,000 ft, -40°C to ISA+35°C, validated by dynamometer and flight test (multi-dimensional, temperature and altitude interact)
- Flight envelope: VMCA to VMO/MMO, validated across CG range and weight range (multi-dimensional with interaction: allowable speed range depends on weight and CG)
- Manufacturing process capability: Cpk ≥ 1.67 for turbine blade root form grinding at ±0.005mm (scalar)
- Ground system SLA: p99 latency ≤ 200ms validated at up to 50,000 concurrent connections (scalar)
- Battery discharge envelope: validated from -40°C to 55°C, 10%-100% SoC, with interaction constraint that discharge rate is coupled to temperature (multi-dimensional with interaction)
- Multi-axis operating envelope: validated across temperature, supply voltage, vibration, and altitude with interactions between temperature and altitude affecting component derating (multi-dimensional with interactions)

**Example: scalar envelope.**

```yaml
id: DES-ENV-PROCESS-CPK
shape: envelope
satisfies: [OBL-COMP-GRINDING-CAPABILITY]
components: [COMP-GRINDING-STATION-A]
dimension: blade_root_form_tolerance
validated_range: {min: -0.005, max: 0.005, unit: mm}
characterization_method: "statistical process study"
```

**Example: multi-dimensional envelope with interactions.**

```yaml
id: DES-ENV-OPERATING-RANGE
shape: envelope
satisfies: [OBL-SYS-OPERATING-ENVELOPE]
components: [COMP-CONTROLLER, COMP-ACTUATORS, COMP-SENSORS, COMP-STRUCTURE]
dimensions:
  - {name: ambient_temperature, range: {min: -40, max: 55, unit: celsius}}
  - {name: supply_voltage, range: {min: 18, max: 32, unit: VDC}}
  - {name: payload_mass, range: {min: 0, max: 500, unit: kg}}
  - {name: surface_slope, range: {min: 0, max: 15, unit: degrees}}
  - {name: surface_friction, range: {min: 0.2, max: 1.0, unit: dimensionless}}
interaction_constraints:
  - parameters: [surface_slope, payload_mass]
    expression: "surface_slope * payload_mass <= 5000"
    rationale: "Increased load reduces slope tolerance due to CG shift"
  - parameters: [surface_friction, surface_slope]
    expression: "surface_slope <= 12 * surface_friction"
    rationale: "Slip risk couples friction and slope"
  - parameters: [ambient_temperature, supply_voltage]
    expression: "NOT (ambient_temperature > 45 AND supply_voltage < 22)"
    rationale: "Thermal derating reduces voltage margin at high temperature"
representation: hyperrectangle_with_interactions
characterization_method: "test + analysis"
characterization_points: 847
constraints:
  - {type: CT-CONTAINMENT-ND, metric: operating_envelope,
     operator: "contains",
     value: [OC-TEMP, OC-VOLTAGE, OC-PAYLOAD, OC-SLOPE, OC-FRICTION]}
verification_plan:
  - parameter: operating_envelope
    method: test
    rationale: "Physical tests at envelope corners and critical interaction boundaries"
  - parameter: operating_envelope
    method: analysis
    rationale: "Simulation sweep with validated plant model across parameter space"
```

NOTE: An Envelope design MAY be referenced by different obligations in different mode states (see Section 5.4 and Rule 7.10). This models the pattern where validated capability boundaries change with system state (for example, reduced operating range under degraded conditions, tighter process tolerances during machine warm-up, or restricted flight envelope after partial system failure).

**Typical constraint types:** containment, containment-nd, threshold, stability.

### 5.4 Mode

Models valid states, transitions, guards, and invariants for a stateful element.

|Field              |Type                              |Required|Description                                                                                                  |
|-------------------|----------------------------------|--------|-------------------------------------------------------------------------------------------------------------|
|states             |list<State>                       |yes     |Valid states. Each has: name, invariants (obligations that apply in this state), entry_actions, exit_actions.|
|transitions        |list<Transition>                  |yes     |Valid transitions. Each has: source, target, trigger, guard (condition that must hold), actions.             |
|initial_state      |string                            |yes     |The starting state.                                                                                          |
|mode_obligation_map|map<StateName, list<ObligationID>>|no      |Which obligations apply in which states. Obligations not in the map apply in ALL states. See Rule 7.10.     |

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
|resource     |string          |yes     |What is budgeted: thrust, weight, power, compute, fuel_flow, bandwidth, cost, cycle_time, thermal_dissipation, etc.|
|unit         |string          |yes     |Unit of measurement.                                                                                               |
|supply       |number          |yes     |Total available quantity.                                                                                          |
|allocations  |list<Allocation>|yes     |Per-component demands. Each has: component, demand, priority (optional).                                           |
|margin_policy|MarginPolicy    |no      |Required reserve: {minimum_margin_percent, rationale}.                                                             |

**Domain examples:**

- Weight budget: 85,000 kg MTOW allocated across airframe (22,000), engines (7,000), systems (8,000), payload (15,000), fuel (28,000), margin (5,000)
- Electrical power budget: 150 kVA total from generators, allocated across avionics (40), flight controls (30), galley (25), environmental (35), de-icing (15), margin (5)
- Thrust budget: 2 × 120 kN available, allocated for climb gradient, acceleration, and bleed air extraction
- Avionics compute budget: allocated across flight management (30%), display processing (25%), engine monitoring (20%), comms (15%), margin (10%)
- Manufacturing cycle time: 120s per blade, allocated across rough grinding (40s), finish grinding (35s), inspection (25s), deburring (20s)

**Typical constraint types:** balance, threshold.

### 5.6 Open Shape Membership

The five shapes above represent relationships encountered in the systems examined to date. This set is explicitly open. A new shape MAY be added when ALL of the following conditions are met:

**Rule 5.6.1.** The relationship requires validation logic that cannot be expressed using any existing shape's constraint types and structure.

**Rule 5.6.2.** The new shape has at least one constraint type that is structurally distinct from all constraint types used by existing shapes, OR applies existing constraint types in a structurally different validation pattern.

**Rule 5.6.3.** The new shape is documented with: name, relationship modeled, shape-specific fields, applicable constraint types, at least three domain examples spanning at least two engineering disciplines, and a justification that existing shapes cannot model the relationship.

Adding a shape does not invalidate existing declarations. The derivation pattern, composition rules, and gap analysis extend to new shapes without modification.

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

The acceptance function evaluates evidence at the *design* constraint level. Because the design constraint refines the obligation constraint (Rule 7.3), evidence satisfying the design's acceptance function necessarily satisfies the obligation's constraint by transitivity.

### 6.2 Constraint Structure and Measurement Payload

Every constraint instance has:

|Field   |Type                    |Required|Description                             |
|--------|------------------------|--------|----------------------------------------|
|type    |ConstraintTypeID        |yes     |Which constraint type.                  |
|metric  |string                  |yes     |What is being measured.                 |
|operator|string                  |yes     |The comparison operator (type-specific).|
|value   |number, string, or range|yes     |The threshold, target, or specification.|
|unit    |string                  |yes     |Unit of measurement.                    |

Every evidence measurement payload has:

|Field |Type                              |Required|Description                                                                                                        |
|------|----------------------------------|--------|-------------------------------------------------------------------------------------------------------------------|
|metric|string                            |yes     |What was measured. Must match the constraint metric.                                                               |
|value |number, string, range, or sequence|yes     |The measured, observed, or assessed result. Type must be compatible with the constraint type's acceptance function.|
|unit  |string                            |yes     |Unit of measurement. Must match the constraint unit.                                                               |

The acceptance function for a constraint type takes a measurement payload and returns `pass`, `fail`, or `inconclusive`. The `inconclusive` result is reserved for cases where the measurement is valid but the acceptance function cannot determine satisfaction (e.g., a measurement at the exact boundary of a threshold).

**Statistical constraint additional fields:** CT-STATISTICAL constraints extend the base constraint structure with:

|Field              |Type   |Required|Description                                                                                               |
|-------------------|-------|--------|----------------------------------------------------------------------------------------------------------|
|statistic          |enum   |yes     |The summary statistic: `p50`, `p90`, `p95`, `p99`, `p999`, `mean`, `cpk`, `ppk`, `sigma_level`, `std_dev`.|
|sample_size_minimum|integer|no      |Minimum number of observations required for a valid evaluation.                                           |
|observation_window |string |no      |Time window over which observations are collected (e.g., `3600s`, `24h`).                                 |
|confidence_level   |float  |no      |Required confidence level for the statistic (e.g., 0.95).                                                 |

**Statistical measurement payload additional fields:**

|Field             |Type   |Required|Description                                                                     |
|------------------|-------|--------|--------------------------------------------------------------------------------|
|statistic         |enum   |yes     |Which statistic was computed. Must match the constraint statistic.              |
|computed_value    |number |yes     |The computed statistic value. This is evaluated by the acceptance function.     |
|sample_size       |integer|yes     |Number of observations in the sample. Must meet sample_size_minimum if declared.|
|observation_window|number |no      |Actual observation window in seconds.                                           |

**N-dimensional containment constraint additional fields:** CT-CONTAINMENT-ND constraints extend the base constraint structure with:

|Field                  |Type                       |Required|Description                                                                                                                 |
|-----------------------|---------------------------|--------|----------------------------------------------------------------------------------------------------------------------------|
|dimensions             |list<DimensionSpec>        |yes     |The dimensions of the required region. Each has: name, range ({min, max, unit}).                                            |
|interaction_constraints|list<InteractionConstraint>|no      |Constraints on joint relationships between dimensions. Each has: parameters, expression, rationale. See Section 5.3.        |
|representation         |enum                       |no      |One of: `hyperrectangle`, `hyperrectangle_with_interactions`, `convex_hull`, `parametric_boundary`. Default: inferred from structure.|

**N-dimensional containment refinement rule:**

The design's validated region MUST contain the obligation's required region. Containment checking depends on the representations used:

- **Both hyperrectangles (with or without interactions):** Containment holds if every vertex of the obligation's required region (intersected with its interaction constraints) is contained within the design's validated region (intersected with its interaction constraints). For a required region in d dimensions with interaction constraints expressible as linear inequalities, this is a finite set of linear programs.

- **Obligation hyperrectangle, design convex hull:** Containment holds if every vertex of the obligation's required region is inside the design's convex hull. Point-in-hull is a linear program with N constraints (N = number of hull-defining points) and d variables.

- **Obligation hyperrectangle, design parametric boundary:** Containment holds if the minimum of the boundary function over the obligation's required region meets or exceeds the threshold. This is a bounded optimization problem. If the boundary function is concave over the required region, the minimum is at a vertex.

For all representations, the containment check is decidable and tractable for the dimensionalities typical of engineered systems (d ≤ 20).

**N-dimensional containment acceptance function:**

An evidence point (a measurement in ℝⁿ at specific operating condition values) passes if the point falls within the design's validated region. For hyperrectangles with interactions: the point must satisfy all per-dimension range checks AND all interaction constraints. For convex hulls: point-in-hull test. For parametric boundaries: function evaluation at the point must meet or exceed the threshold.

**Relationship between CT-CONTAINMENT and CT-CONTAINMENT-ND:**

CT-CONTAINMENT is the single-dimensional case. CT-CONTAINMENT-ND is the general multi-dimensional case. A CT-CONTAINMENT constraint is semantically equivalent to a CT-CONTAINMENT-ND constraint with one dimension and no interaction constraints. Implementations MAY treat CT-CONTAINMENT as syntactic shorthand for the single-dimensional CT-CONTAINMENT-ND case.

**Example constraints and corresponding evidence:**

```yaml
# Constraint (on design)
type: CT-TIMING
metric: engine_failure_to_annunciation
operator: "<="
value: 50
unit: ms

# Evidence measurement payload
metric: engine_failure_to_annunciation
value: 42
unit: ms
# Acceptance: 42 <= 50 → pass
```

```yaml
# Constraint (on design)
type: CT-THRESHOLD
metric: ac_bus_voltage
operator: "within"
value: {min: 108, max: 118}
unit: VAC

# Evidence measurement payload
metric: ac_bus_voltage
value: 115.2
unit: VAC
# Acceptance: 115.2 ∈ [108, 118] → pass
```

```yaml
# Constraint (on design)
type: CT-MATCH
metric: mount_bolt_spec
operator: "="
value: "NAS6204-26"
unit: part_number

# Evidence measurement payload (from inspection)
metric: mount_bolt_spec
value: "NAS6204-26"
unit: part_number
# Acceptance: "NAS6204-26" = "NAS6204-26" → pass
```

```yaml
# Constraint (on design)
type: CT-BALANCE
metric: max_takeoff_weight
operator: "sum_lte"
value: 85000
unit: kg

# Evidence measurement payload
metric: max_takeoff_weight
value: {supply: 85000, demands: [22000, 7000, 8000, 15000, 28000]}
unit: kg
# Acceptance: Σ(80000) ≤ 85000 → pass
```

```yaml
# Constraint (on design)
type: CT-STATISTICAL
metric: blade_root_form_cpk
operator: ">="
value: 1.67
unit: dimensionless
statistic: cpk
sample_size_minimum: 30

# Evidence measurement payload
metric: blade_root_form_cpk
value: {statistic: cpk, computed_value: 1.82, sample_size: 50,
        sample_mean: 0.0002, sample_std: 0.0009}
unit: dimensionless
# Acceptance: 1.82 >= 1.67 → pass
```

```yaml
# Constraint (on design)
type: CT-STATISTICAL
metric: api_response_latency_p99
operator: "<="
value: 200
unit: ms
statistic: p99
observation_window: 3600s

# Evidence measurement payload
metric: api_response_latency_p99
value: {statistic: p99, computed_value: 187, sample_size: 48230,
        observation_window: 3600}
unit: ms
# Acceptance: 187 <= 200 → pass
```

```yaml
# Constraint (on obligation): N-dimensional containment
type: CT-CONTAINMENT-ND
metric: operating_envelope
operator: "contains"
dimensions:
  - {name: ambient_temperature, range: {min: -20, max: 50, unit: celsius}}
  - {name: supply_voltage, range: {min: 20, max: 30, unit: VDC}}
  - {name: payload_mass, range: {min: 0, max: 400, unit: kg}}
interaction_constraints:
  - parameters: [ambient_temperature, supply_voltage]
    expression: "NOT (ambient_temperature > 40 AND supply_voltage < 24)"
    rationale: "Thermal derating reduces voltage margin"

# Constraint (on design): wider validated region
type: CT-CONTAINMENT-ND
metric: operating_envelope
operator: "contains"
dimensions:
  - {name: ambient_temperature, range: {min: -40, max: 55, unit: celsius}}
  - {name: supply_voltage, range: {min: 18, max: 32, unit: VDC}}
  - {name: payload_mass, range: {min: 0, max: 500, unit: kg}}
interaction_constraints:
  - parameters: [ambient_temperature, supply_voltage]
    expression: "NOT (ambient_temperature > 40 AND supply_voltage < 22)"
    rationale: "Thermal derating: design limit stricter than obligation"
# Refinement: design region ⊇ obligation region on all dimensions
# and design interaction constraint is at least as strict. ✓

# Evidence measurement payload: single operating point
metric: operating_envelope
value: {ambient_temperature: 35, supply_voltage: 26, payload_mass: 350}
unit: multi_dimensional_point
# Acceptance: point within design ranges AND satisfies all
# interaction constraints → pass
```

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

**Rule 7.1.1.** Every Obligation with status `active` MUST reference exactly one parent, which is either a Requirement ID or a parent Obligation ID.

**Rule 7.1.2.** Every Obligation with status `active` MUST be reachable from at least one Requirement by traversing parent references upward. No orphan obligations.

**Rule 7.1.3.** Every Design MUST reference at least one Obligation with status `active` in its `satisfies` field.

**Rule 7.1.4.** The parent reference graph of obligations MUST be acyclic (a directed acyclic graph).

### 7.2 Scope Consistency

**Rule 7.2.1.** An obligation with scope `intrinsic` MAY be satisfied by a design involving one or more components (the constraint applies to a single component's property, but the design may involve context).

**Rule 7.2.2.** An obligation with scope `relational` MUST be satisfied by a design involving at least two components and referencing an interface.

**Rule 7.2.3.** An obligation with scope `emergent` MUST be satisfied by a design whose components collectively span the system boundary relevant to the emergent property.

### 7.3 Refinement

**Rule 7.3.1.** For every obligation O referenced in a design D's `satisfies` list, D MUST declare at least one constraint of the same type as O's constraint.

**Rule 7.3.2.** Each such design constraint MUST be at least as strict as the corresponding obligation constraint, as defined by the refinement rule for that constraint type (Section 6.1).

**Rule 7.3.3.** If a design constraint is not at least as strict as its obligation constraint, the declaration set is non-conformant and the specific violation MUST be reported.

**Rule 7.3.4.** Refinement establishes transitivity of evidence: evidence satisfying a design's constraint necessarily satisfies the obligation's constraint, because the design's constraint is at least as strict.

### 7.4 Obligation Decomposition Consistency

**Rule 7.4.1.** If an obligation O has children C₁, C₂, …, Cₙ, the children's constraints MUST be consistent with O's constraint as follows:

- For **timing** obligations: Σ(child timing values) ≤ parent timing value (timing budgets close).
- For **balance** obligations: Σ(child supply values) ≤ parent supply value AND child allocations partition the parent resource.
- For **match** obligations: all children must reference the same matched value.
- For **threshold** obligations: each child's range must be contained within the parent's range.
- For **containment** obligations (CT-CONTAINMENT and CT-CONTAINMENT-ND): child validated regions, projected onto the parent's dimensions, must collectively cover the parent's required region. For CT-CONTAINMENT-ND, the intersection of child envelopes across shared dimensions must contain the parent's required region including its interaction constraints.
- For other types: the composition rule is type-specific. If no explicit composition rule is defined for a constraint type, the parent obligation MUST include in its verification plan an `analysis` or `inspection` method entry covering the composition assertion. This evidence, produced through the parent's satisfying design, discharges the composition proof obligation.

**Rule 7.4.2.** If an obligation O has children, the union of the children's scopes and components MUST cover all components and interfaces relevant to O. This is the *completeness* property. Incomplete decomposition is a structural warning (not a conformance violation) that MUST be reported.

### 7.5 Cross-Obligation Consistency

**Rule 7.5.1.** If two obligations O₁ and O₂ with status `active` apply to the same component and have constraints of the same type on the same metric, the constraints MUST NOT be contradictory. Example: O₁ requires `engine_mode = max_continuous_thrust` and O₂ requires `engine_mode = idle` during the same flight phase is a contradiction.

**Rule 7.5.2.** Contradictory obligations MUST be reported as structural errors. Resolution requires either modifying one obligation or introducing a mode design that separates the contexts in which each applies.

### 7.6 Safety Obligations

**Rule 7.6.1.** An obligation derived (directly or transitively) from a requirement with `source_type = safety_analysis` is a *safety obligation*.

**Rule 7.6.2.** Safety obligations MUST NOT appear in any operating condition's exemption list. Safety obligations apply under all declared conditions.

**Rule 7.6.3.** Safety obligations SHOULD declare `min_verification_fidelity` of `HIL` or higher. This is a recommendation, not a conformance requirement, as the appropriate fidelity depends on the safety integrity level and applicable standard (ARP4754A, DO-178C, DO-254).

**Rule 7.6.4.** Safety obligations MUST NOT have status `withdrawn` without reference to a revised safety assessment that removes the underlying hazard or modifies its severity classification. This reference MUST be recorded in a rationale field.

### 7.7 Operating Conditions

**Rule 7.7.1.** Every operating condition applies to ALL obligations with status `active` by default.

**Rule 7.7.2.** An obligation MAY declare specific exemptions from non-safety operating conditions. Each exemption MUST include a rationale.

**Rule 7.7.3.** The cross-product of obligations and applicable operating conditions defines the *verification matrix*. Each cell in the matrix represents a required piece of evidence.

### 7.8 Component and Interface References

**Rule 7.8.1.** Every Component ID and Interface ID referenced by any declaration MUST exist in the World model. The World model is the set of declared components and interfaces.

**Rule 7.8.2.** An interface MUST reference exactly two components.

### 7.9 Verification Plan Coverage

**Rule 7.9.1.** Every obligation MUST declare a verification plan with at least one entry.

**Rule 7.9.2.** Every design MUST declare a verification plan with at least one entry.

**Rule 7.9.3.** For each obligation O in a design D's `satisfies` list, every parameter in O's verification plan MUST be covered by at least one entry in D's verification plan with the same verification method. This is the *verification plan coverage* property. Missing coverage is a structural conformance violation.

### 7.10 State-Dependent Obligation Activation

**Rule 7.10.1.** If a Mode design declares a `mode_obligation_map`, then for each state S in the mode, the obligations mapped to S MUST be structurally conformant as an independent subset: traceability (Rule 7.1), scope consistency (Rule 7.2), refinement (Rule 7.3), and cross-obligation consistency (Rule 7.5) MUST hold within each state's obligation set.

**Rule 7.10.2.** Obligations not appearing in any `mode_obligation_map` entry apply in ALL states of that mode. These are *state-invariant* obligations.

**Rule 7.10.3.** Obligations appearing in a `mode_obligation_map` entry apply ONLY in the listed states. Evidence for state-specific obligations is required only under operating conditions applicable during that state.

**Rule 7.10.4.** Cross-obligation consistency (Rule 7.5) is checked per-state. Two obligations that would be contradictory if both active simultaneously are permitted if they are mapped to non-overlapping states. This is the primary mechanism for resolving contradictions identified by Rule 7.5: introduce a mode design that separates the contexts.

**Rule 7.10.5.** When a Mode design's `mode_obligation_map` activates different Envelope obligations in different states, the envelope applicable in each state defines the validated operating boundary for that state. The system MUST operate within the envelope corresponding to its current mode state.

**Rule 7.10.6.** Gap analysis and verdict computation for state-specific obligations operate within the scope of each state. An obligation mapped to state S requires evidence only under the operating conditions applicable during state S. The verification matrix for the system is the union of the per-state verification matrices.

NOTE: State-dependent obligation activation is the general mechanism for modeling systems where constraints change with operating phase. Examples include: reduced performance envelopes under degraded hardware conditions, tighter tolerances during critical manufacturing phases, restricted operating ranges during system warm-up or cool-down, different SLA thresholds during failover versus normal operation, and reduced capability envelopes following partial system failures.

-----

## 8. Composition Rules

Composition is the process by which independently verified child obligations combine to satisfy a parent obligation.

### 8.1 Basic Composition

**Rule 8.1.1.** A parent obligation O is *satisfied by composition* if and only if ALL of the following hold:

(a) Every child obligation Cᵢ of O with status `active` has status `satisfied` (all applicable operating conditions have passing evidence at required fidelity, resolved through each child's satisfying designs).

(b) The decomposition is *complete*: the set of child obligations covers all facets of the parent obligation (Rule 7.4.2).

(c) The decomposition is *consistent*: child constraints compose correctly to the parent constraint (Rule 7.4.1).

(d) The parent obligation O itself has evidence at its own declared fidelity, resolved through its own satisfying design(s). Composition is necessary but not sufficient: child evidence demonstrates that parts work; parent-level evidence demonstrates that the composed whole exhibits the required behavior.

### 8.2 Cross-Shape Composition

**Rule 8.2.1.** A parent obligation MAY be decomposed into children satisfied by designs of different shapes. Each shape validates independently using its own constraint types and harnesses.

**Rule 8.2.2.** Completeness for cross-shape decomposition is checked at the parent obligation: does the set of designs, across all shapes, account for every facet of the parent obligation?

**Rule 8.2.3.** Consistency for cross-shape decomposition requires that shared parameters agree across shapes. If a climb-gradient obligation decomposes into a budget design (thrust minus drag) and an envelope design (validated airspeed range), the airspeed assumptions in both designs must reference the same operating range and their values must be consistent.

**Example:** System obligation "maintain ≥ 2.4% climb gradient OEI" decomposes into:

|Child Obligation                                               |Shape   |Constraint|
|---------------------------------------------------------------|--------|----------|
|Remaining engine thrust ≥ 105 kN at takeoff                    |envelope|threshold |
|Drag budget closes with one engine failed                      |budget  |balance   |
|Flight control compensates asymmetric thrust within 3s         |exchange|timing    |
|FADEC detects failure and configures remaining engine within 1s|exchange|timing    |

Consistency check: thrust (105 kN) minus drag (at declared weight and configuration) yields ≥ 2.4% gradient. Budget closes.
Completeness check: all contributing factors to climb gradient are covered (thrust, drag, control response, engine reconfiguration).

### 8.3 Composition Across Fidelity

**Rule 8.3.1.** Evidence produced at fidelity level F satisfies any obligation requiring fidelity ≤ F. Higher-fidelity evidence is always acceptable.

**Rule 8.3.2.** A parent obligation's evidence requirement is independent of its children's fidelity levels. The parent declares its own `min_verification_fidelity` and requires evidence at that fidelity resolved through its own satisfying design. Child evidence at any fidelity does not substitute for parent-level evidence. This ensures that composed system behavior is verified at the system level, not merely inferred from component-level results.

### 8.4 Monotonicity and Obligation Lifecycle

**Rule 8.4.1.** Active obligations are append-only. An obligation with status `active`, once declared, MUST NOT be removed from the declaration set. Its status may transition as follows:

- `active` → `superseded`: Replaced by a stricter obligation. The superseding obligation MUST be identified. All designs referencing the superseded obligation MUST be updated to reference the superseding obligation or re-verified.
- `active` → `withdrawn`: No longer applicable due to scope change. MUST include a rationale. MUST trigger re-evaluation of all parent obligations that depend on it. Safety obligations have additional restrictions (Rule 7.6.4).

Obligations with status `superseded` or `withdrawn` remain in the declaration graph for auditability but are excluded from gap analysis and verdict computation.

**Rule 8.4.2.** Evidence is append-only. Evidence, once produced, MUST NOT be removed. It may be superseded by more recent evidence.

**Rule 8.4.3.** Relaxation of an obligation constraint (making it less strict) MUST be accompanied by a rationale and MUST trigger re-evaluation of all designs that reference the obligation and all parent obligations that depend on it.

-----

## 9. Gap Analysis

Gap analysis computes the difference between what should be verified and what has been verified.

### 9.1 Gap Types

|Gap Type      |Detection Rule                                                                                                |Nature                       |Resolution                          |
|--------------|--------------------------------------------------------------------------------------------------------------|-----------------------------|------------------------------------|
|Decomposition |A requirement R has `decomposed_into` = empty                                                                 |Knowledge (ongoing)          |Analyze R and create obligations    |
|Scenario      |A field observation, manufacturing nonconformance, or test failure cannot be mapped to any existing obligation|Knowledge (ongoing)          |Create new obligation from discovery|
|Infrastructure|An obligation O exists but no harness is registered for O's medium or domain                                  |Tooling (one-time per medium)|Implement harness for the medium    |
|Observability |An obligation O exists but no TelemetryMapping exists for O's metric                                          |Tooling (one-time per metric)|Implement telemetry mapping         |

### 9.2 Gap Computation

Gap analysis operates only on obligations with status `active`.

**Rule 9.2.1.** For each requirement R: if R.decomposed_into is empty or all referenced obligations have status `withdrawn`, emit a decomposition gap for R.

**Rule 9.2.2.** For each active obligation O, for each entry in O's verification plan, for each applicable operating condition C: if no evidence exists at the satisfying design(s) for that (parameter, method, condition) tuple at fidelity ≥ O.min_verification_fidelity, emit an evidence gap. Classify the gap:

- If no harness exists for O's domain → infrastructure gap.
- If no TelemetryMapping exists for O's metric → observability gap.
- Otherwise → test execution gap (evidence has not yet been produced but could be).

**Rule 9.2.3.** For each field observation, manufacturing nonconformance, or incident report: if the observation falls outside all obligation envelopes, emit a scenario gap.

**Rule 9.2.4.** Runtime observations that satisfy an obligation's acceptance function under a specific operating condition, produced at field fidelity, MAY be ingested as field-fidelity evidence. Such evidence is subject to the same acceptance function evaluation and constraint type matching as design-time evidence. For CT-CONTAINMENT-ND obligations, a runtime observation at an operating point within the validated region constitutes field-fidelity evidence for that specific point in the condition space. The ingestion mechanism, including criteria for observation quality, measurement uncertainty, and minimum observation count, is an implementation concern outside this specification.

NOTE: Rule 9.2.4 connects the runtime monitoring tier (RuntimeConstraint, TelemetryMapping) to the evidence accumulation lifecycle. Field observations that pass the acceptance function expand the set of operating conditions with direct evidence. Field observations that fail the acceptance function or fall outside the declared operating envelope produce scenario gaps (Rule 9.2.3). Together, these mechanisms support monotonic gap closure: successful field operation produces evidence, and unsuccessful or out-of-envelope operation produces gaps that drive obligation expansion.

### 9.3 Gap Metrics

The following metrics are computed from gap analysis:

- **Obligation coverage:** (active obligations with full evidence across all verification plan entries) / (total active obligations)
- **Condition coverage:** (obligation × parameter × method × condition tuples with evidence) / (total required tuples)
- **Decomposition completeness:** (requirements with ≥ 1 active obligation) / (total requirements)
- **Infrastructure readiness:** (active obligations with available harness) / (total active obligations)
- **Observability readiness:** (active obligations with TelemetryMapping) / (total active obligations)

-----

## 10. Verification Environments

### 10.1 Fidelity Hierarchy

The following fidelity levels are ordered from lowest to highest:

```
bench < SIL < HIL < system_test < field
```

**Rule 10.1.1.** Evidence at fidelity F is acceptable for any obligation with min_verification_fidelity ≤ F.

**Rule 10.1.2.** The fidelity hierarchy is fixed. New fidelity levels MUST NOT be inserted between existing levels. Domain-specific sub-levels (e.g., distinguishing between different HIL configurations) are modeled as distinct VerificationEnvironment instances sharing the same fidelity level.

### 10.2 Fidelity Requirements

**Rule 10.2.1.** If an obligation has no `min_verification_fidelity` declared, the default is `bench` (component-level isolation testing is the minimum acceptable evidence).

**Rule 10.2.2.** Obligations derived from safety requirements SHOULD declare `min_verification_fidelity` of `HIL` or higher. The appropriate level depends on the safety integrity level and applicable standard (ARP4754A DAL A-D, DO-178C, DO-254).

**Rule 10.2.3.** The verification matrix for a release is the cross-product of: active obligations × applicable operating conditions × verification plan entries × required verification fidelities. Each cell requires evidence resolved through the obligation's satisfying designs.

-----

## 11. Extensibility

### 11.1 Extension Categories

OBSA has three categories of extensibility:

|Category          |Governance                                                                |Examples                                                                                                                                                                                 |
|------------------|--------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Fixed core        |Not extensible                                                            |Declaration hierarchy (Requirement → Obligation → Design), structural rules, composition rules, gap types, fidelity hierarchy, verification methods (TIAD), evidence-through-design model|
|Governed extension|Extensible with documented justification per rules in Sections 5.6 and 6.3|Design shapes, constraint types                                                                                                                                                          |
|Free instantiation|Freely extensible by any user                                             |Obligations, designs, operating conditions, components, interfaces, harnesses, adapters, telemetry mappings                                                                              |

### 11.2 Domain Extension

**Rule 11.2.1.** Extending OBSA to a new domain requires only harnesses and adapters for that domain's interface media and verification tools. The declaration hierarchy, structural rules, composition rules, and gap analysis are domain-invariant.

**Rule 11.2.2.** A domain extension MUST NOT modify the structural rules, composition rules, or conformance criteria.

### 11.3 Backward Compatibility

**Rule 11.3.1.** Adding a new shape or constraint type MUST NOT invalidate any existing conformant declaration set.

**Rule 11.3.2.** New shapes and constraint types are additive. Existing declarations continue to be valid under their original shapes and constraint types.

### 11.4 Configuration Management

Each declaration set represents a single system configuration. Multi-configuration management (e.g., fleet variants, software versions coexisting in production) is handled at the instantiation layer through standard version control and configuration management practices. The structural rules, particularly Rule 8.4.3, govern when evidence must be re-evaluated after changes to obligations or designs within a declaration set.

-----

## 12. Conformance

### 12.1 Conformance Levels

OBSA defines two conformance levels:

**Level 1: Structurally Conformant.** A declaration set is structurally conformant if and only if all rules in Section 7 are satisfied. The declaration set is well-formed. Designs refine obligations. Obligations trace to requirements. The DAG is acyclic. Cross-obligation consistency holds. Verification plans are complete. State-dependent obligation activation is consistent (Rule 7.10). This level can be checked mechanically from the declaration set alone, without executing any tests.

**Level 2: Evidence-Complete.** A declaration set is evidence-complete if and only if:
(a) It is structurally conformant (Level 1).
(b) Every active obligation has passing evidence resolved through its satisfying designs, at required fidelity, under all applicable operating conditions, for all verification plan entries. For state-specific obligations (Rule 7.10), evidence is required only under the operating conditions applicable during the obligation's mapped states.
(c) All composition rules (Section 8) are satisfied for every parent obligation, including parent-level evidence (Rule 8.1.1(d)).
(d) No decomposition gaps exist (every requirement has at least one active obligation).

### 12.2 Conformance Checking

**Rule 12.2.1.** Level 1 conformance is checkable from the declaration set alone. No test execution required. This includes verification plan coverage checking (Rule 7.9) and state-dependent obligation consistency checking (Rule 7.10).

**Rule 12.2.2.** Level 2 conformance requires the declaration set plus an evidence set. The evidence set is compared against the verification matrix (active obligations × conditions × verification plan entries × fidelities). Evidence is located at the design level and evaluated through constraint type acceptance functions.

**Rule 12.2.3.** A conformance report MUST enumerate:

- All structural rule violations (Level 1 failures).
- All verification plan coverage gaps (obligation parameters not covered by design verification plans).
- All evidence gaps with classification (Level 2 gaps).
- All composition failures with specific parent obligations and failing conditions.
- All state-dependent obligation consistency violations (Rule 7.10 failures).
- Coverage metrics per Section 9.3.

### 12.3 Partial Conformance

In practice, systems achieve conformance incrementally. A declaration set may be structurally conformant (Level 1) while working toward evidence-complete (Level 2). Gap analysis (Section 9) provides the roadmap from Level 1 to Level 2.

-----

## 13. Examples

### 13.1 Cross-Domain Composition Example

This example demonstrates a system-level obligation decomposing through multiple shapes, domains, and fidelity levels for a transport aircraft one-engine-inoperative (OEI) continued takeoff scenario.

**Requirement:**

```yaml
id: REQ-SAFE-001
title: Single Engine Continued Takeoff
source_type: safety_analysis
source_reference: FHA-2025-003, Hazard H7; 14 CFR 25.121(a)
text: "The aircraft shall maintain a positive climb gradient of not less
       than 2.4% following a single engine failure recognized at or above
       V1 during takeoff with landing gear retracted."
decomposed_into: [OBL-SYS-OEI-CLIMB]
```

**System obligation:**

```yaml
id: OBL-SYS-OEI-CLIMB
title: OEI Climb Gradient at Second Segment
parent: REQ-SAFE-001
source_type: requirement_decomposition
scope: emergent
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
  - parameter: aerodynamic_model_correlation
    method: analysis
    rationale: "CFD model validated against wind tunnel and flight test"
  - parameter: pilot_engine_failure_recognition
    method: demonstration
    rationale: "Witnessed simulator evaluation per AC 25-7D"
min_verification_fidelity: system_test
children:
  - OBL-SUB-ENGINE-OEI-THRUST
  - OBL-SUB-DRAG-CLEAN-CONFIG
  - OBL-SUB-FCS-ASYMMETRY-COMP
  - OBL-SUB-FADEC-RECONFIG
```

**Subsystem obligations (four children, different shapes):**

```yaml
# Child 1: Remaining engine available thrust (Envelope shape)
id: OBL-SUB-ENGINE-OEI-THRUST
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: intrinsic
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
min_verification_fidelity: bench
satisfied_by: [DES-ENV-ENGINE-THRUST]

# Child 2: Aircraft drag budget in clean configuration (Budget shape)
id: OBL-SUB-DRAG-CLEAN-CONFIG
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: emergent
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
min_verification_fidelity: SIL
satisfied_by: [DES-BDG-DRAG]

# Child 3: Flight control asymmetric thrust compensation (Exchange shape)
id: OBL-SUB-FCS-ASYMMETRY-COMP
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: relational
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
    rationale: "Measured response on HIL rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case execution time analysis"
min_verification_fidelity: HIL
satisfied_by: [DES-EXCH-FCS-FADEC]

# Child 4: FADEC failure detection and engine reconfiguration (Exchange shape)
id: OBL-SUB-FADEC-RECONFIG
parent: OBL-SYS-OEI-CLIMB
source_type: requirement_decomposition
scope: intrinsic
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
    rationale: "Fault injection test on HIL rig"
min_verification_fidelity: HIL
satisfied_by: [DES-EXCH-FADEC-CMD]
```

**Designs (four shapes):**

```yaml
# Envelope design: engine thrust characterization
id: DES-ENV-ENGINE-THRUST
shape: envelope
title: Engine Thrust Envelope Characterization
satisfies: [OBL-SUB-ENGINE-OEI-THRUST]
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
supply: 118    # single engine available thrust at MTOW, ISA+15
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
interface_id: IF-FCS-FADEC
medium: ARINC_429
flows:
  - {name: thrust_asymmetry_command, direction: out,
     format: ARINC_429_label_312, rate: 50Hz}
  - {name: engine_status, direction: in,
     format: ARINC_429_label_200, rate: 20Hz}
constraints:
  - {type: CT-TIMING, metric: asymmetry_compensation_time,
     operator: "<=", value: 2.5, unit: s}
  # 2.5s design < 3s obligation: refinement satisfied.
verification_plan:
  - parameter: asymmetry_compensation_time
    method: test
    rationale: "ARINC 429 bus timing measurement on HIL rig"
  - parameter: asymmetry_compensation_time
    method: analysis
    rationale: "Worst-case execution time analysis of FCS task scheduling"

# Exchange design: FADEC internal failure detection and reconfiguration
id: DES-EXCH-FADEC-CMD
shape: exchange
title: FADEC Cross-Channel Failure Detection
satisfies: [OBL-SUB-FADEC-RECONFIG]
components: [COMP-FADEC-L, COMP-FADEC-R]
interface_id: IF-FADEC-XLINK
medium: ARINC_664
constraints:
  - {type: CT-TIMING, metric: failure_detect_to_reconfig,
     operator: "<=", value: 0.8, unit: s}
  # 0.8s design < 1s obligation: refinement satisfied.
verification_plan:
  - parameter: failure_detect_to_reconfig
    method: test
    rationale: "Fault injection test on HIL rig with ARINC 664 crosslink"
```

**Composition check:**

- **Refinement:** All four designs have constraints stricter than their obligations. ✓
  - Engine thrust: 118 kN ≥ 105 kN
  - Net thrust: 25 kN ≥ 0 kN (actual gradient ~2.9% > 2.4%)
  - Asymmetry compensation: 2.5s ≤ 3s
  - FADEC reconfiguration: 0.8s ≤ 1s
- **Verification plan coverage:** All obligation verification plan parameters are covered by corresponding design verification plan entries with matching methods. ✓
- **Consistency:** Thrust (118 kN) minus drag (93 kN) = 25 kN net, yielding climb gradient above 2.4% at MTOW. Budget closes. ✓
- **Completeness:** All contributing factors covered: available thrust, drag accounting, flight control response, engine reconfiguration. ✓
- **Fidelity:** Parent requires system_test and must have its own evidence at that fidelity. Children require bench, SIL, HIL, HIL respectively. Child evidence does not substitute for parent evidence. ✓
- **Parent evidence:** OBL-SYS-OEI-CLIMB requires evidence at system_test fidelity through its own satisfying design, demonstrating that the composed aircraft actually achieves ≥ 2.4% climb gradient OEI. ✓

**Result:** OBL-SYS-OEI-CLIMB is satisfied by composition if: (1) all four children have passing evidence at their required fidelities under all applicable operating conditions, resolved through their respective designs; AND (2) the parent obligation itself has passing evidence at system_test fidelity, resolved through its own satisfying design, demonstrating the actual exhibited climb gradient under OEI conditions.

-----

## 14. References

- Benveniste, A., et al. (2018). Contracts for System Design. Foundations and Trends in Electronic Design Automation, 12(2-3), 124-400.
- INCOSE. (2023). Systems Engineering Handbook, version 5.0.
- INCOSE. (2022). Guide to Writing Requirements, version 3.1.
- ISO/IEC/IEEE 15288:2023. Systems and Software Engineering — System Life Cycle Processes.
- ISO 26262:2018. Road Vehicles — Functional Safety.
- IEC 61508:2010. Functional Safety of E/E/PE Safety-Related Systems.
- RTCA DO-178C. (2011). Software Considerations in Airborne Systems and Equipment Certification.
- RTCA DO-254. (2000). Design Assurance Guidance for Airborne Electronic Hardware.
- SAE ARP4754A. (2010). Guidelines for Development of Civil Aircraft and Systems.
- SAE ARP4761. (1996). Guidelines and Methods for Conducting the Safety Assessment Process on Civil Airborne Systems and Equipment Certification.
- Meyer, B. (1986). Design by Contract. Technical Report TR-EI-12/CO.
- UL 4600. (2020). Standard for Safety for the Evaluation of Autonomous Products.
- ISO/PAS 8800. (2024). Road Vehicles — Safety and Artificial Intelligence.
- Incer, I. (2022). The Algebra of Contracts. Ph.D. thesis, UC Berkeley.
- ISO 10007:2017. Quality Management — Guidelines for Configuration Management.
- ISO 9001:2015. Quality Management Systems — Requirements.
- AS9100D:2016. Quality Management Systems — Aviation, Space, and Defense.
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

The OBSA declaration structure (typed constraints, refinement relationships, operating condition cross-products, and compositional decomposition) is designed not only for traceability and conformance checking but also to enable intelligent reduction of verification effort. The structure of the declaration set itself contains information that constrains the verification space, reducing the number of tests, simulations, and analyses needed to achieve evidence-complete conformance (Level 2).

This annex describes, informatively, the properties of the declaration structure that support verification effort reduction. The specific algorithms and computational techniques for exploiting these properties are implementation concerns outside the scope of this specification.

### A.2 Monotonicity Exploitation

Many physical and engineered systems exhibit monotonic relationships between operating condition parameters and constraint metrics. When a constraint metric degrades monotonically with increasing (or decreasing) values of an operating condition parameter, evidence at the worst-case extreme of that parameter implies satisfaction across the full range.

If monotonicity can be established (through analysis evidence, physical reasoning, or limited testing), the verification matrix can be reduced to worst-case corners rather than requiring evidence across the full cross-product of operating conditions. For an n-dimensional operating condition space with k monotonic dimensions, the verification points reduce from O(mⁿ) (where m is the number of sample points per dimension) to O(mⁿ⁻ᵏ), yielding exponential savings in the monotonic dimensions.

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

Implementations MAY use child envelope boundaries to identify the critical operating points for parent-level evidence production, focusing system-level tests at conditions where child capabilities limit the parent's performance margin.

### A.6 Multi-Fidelity Evidence Strategies

The fidelity hierarchy (Section 10) provides a framework for combining evidence across fidelity levels:

1. Low-fidelity evidence (bench, SIL) is inexpensive to produce and can cover the parameter space densely. It establishes the general shape of the capability boundary and validates assumptions about monotonicity and interaction structure.

2. High-fidelity evidence (system_test, field) is expensive but authoritative. It anchors the low-fidelity evidence at critical points and satisfies obligations that require higher minimum fidelity.

3. The combination provides coverage confidence: low-fidelity evidence demonstrates that high-fidelity evidence points are representative (not lucky survivors in a region of failures), while high-fidelity evidence provides the authoritative measurements that satisfy obligation constraints.

For multi-dimensional envelopes (CT-CONTAINMENT-ND), this strategy allows dense low-fidelity characterization of the full envelope boundary supported by selective high-fidelity evidence at corners, interaction boundaries, and thin-margin regions. The model validation pattern (Section 4.7.1, informative note) provides the assurance link between the low-fidelity model and the high-fidelity anchor points.

### A.7 Relationship to Verification Effort

The verification matrix (Rule 10.2.3) defines the total evidence requirement: active obligations × applicable operating conditions × verification plan entries × required fidelities. Without exploiting the declaration structure, achieving Level 2 conformance requires evidence for every cell in this matrix.

The properties described in this annex reduce the *effective* verification matrix by identifying cells whose evidence is implied by other cells (monotonicity), cells where evidence production has highest value (margin concentration), cells that satisfy multiple obligations simultaneously (constraint propagation), and regions where low-fidelity evidence provides sufficient coverage confidence (multi-fidelity strategies).

The total verification effort is a function of:

- The size of the verification matrix (determined by the declaration set)
- The structure of the constraints (which enables reduction)
- The available verification environments (which determine cost per evidence point)
- The physical characteristics of the system (which determine monotonicity, interaction structure, and margin distribution)

The OBSA structure provides the information needed for an implementation to compute an efficient verification strategy. The strategy itself (which tests to run, which simulations to execute, which analyses to perform) is an implementation concern that depends on domain knowledge, available resources, and risk tolerance.

-----

## Revision History

| Version | Date | Notes |
|---------|------|-------|
| 0.1.0-draft | 2026-02-18 | Working draft. Added CT-CONTAINMENT-ND constraint type for multi-dimensional envelope support. Added multi-dimensional fields to Envelope design shape (Section 5.3). Added Rule 7.10 for state-dependent obligation activation via mode_obligation_map. Added Rule 9.2.4 for runtime evidence ingestion. Added containment composition rule to Rule 7.4.1. Added model validation informative note to Section 4.7.1. Added Annex A on verification effort reduction. |

-----

## Citation

```
Substrata Labs LLC. (2026). OBSA: Obligation-Based Systems Assurance —
Minimum Viable Specification, Version 0.1.0-draft.
https://github.com/substrata-labs/obsa
```
