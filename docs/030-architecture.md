# Deadline Energy Optimizer

# 030 — Architecture

| Field | Value |
|---|---|
| Specification Version | 0.1.0 |
| Document Version | 1.0 |
| Status | LOCKED |

## 1. Purpose

This document defines the architecture of the Deadline Energy Optimizer (DEO).

It describes the responsibilities, boundaries, data contracts and execution model required to transform the current state of the energy system into a charging decision.

The architecture is intentionally small.

It exists to provide:

- correctness;
- determinism;
- testability;
- local reasoning; and
- clear separation between decision-making and interaction with the outside world.

The detailed requirements are defined in **020 — Requirements**.

The detailed charging algorithm is defined in **040 — Core Algorithm**.

---

## 2. Architectural Principles

### 2.1 Stateless

DEO is stateless.

Each evaluation is independent and operates on the current state of the world supplied to it.

History is state.

The evaluation pipeline does not maintain history between evaluations.

```text
Evaluation N
    │
    ▼
Current SystemState
    │
    ▼
DecisionResult


Evaluation N + 1
    │
    ▼
New SystemState
    │
    ▼
New DecisionResult
```

A previous decision is not an input to the next evaluation unless that information is explicitly part of the current `SystemState`.

---

### 2.2 Deterministic

The evaluation produces the same result when given the same inputs.

There is no hidden state, implicit clock access, random behaviour or external dependency inside the evaluation core.

The current time and all other information required to make a decision are explicit inputs.

---

### 2.3 Immutable

Data passed between transformations is immutable.

A transformation does not modify the data it receives.

Instead, it produces a new immutable output.

```text
Input
  │
  │  unchanged
  ▼
Transformation
  │
  ▼
New Output
```

Immutability simplifies reasoning, testing and failure isolation.

---

### 2.4 Correctness

Correctness is a primary architectural driver.

The system must prefer a clearly correct and understandable design over one that is merely convenient to implement.

The architecture must make it possible to reason about whether a decision is correct from its defined inputs and transformation rules.

---

### 2.5 Testability

The evaluation core is designed so that its transformations can be tested independently.

A transformation can be tested from its input contract to its output contract without requiring the complete system or external infrastructure.

---

### 2.6 Local Reasoning

Each transformation shall be understandable in isolation from its responsibilities, input contract and output contract.

Understanding a transformation shall not require knowledge of the implementation of the whole system.

```text
Known Input
     │
     ▼
Transformation
     │
     ▼
Known Output
```

---

## 3. Architectural Model

The DEO evaluation is a stateless sequence of deterministic transformations over immutable data.

```text
             Outside World
                   │
                   ▼
          Construct SystemState
                   │
                   ▼
               Validate
                   │
                   ▼
            ValidatedState
                   │
                   ▼
                Assess
                   │
                   ▼
          TrustedSystemState
                   │
                   ▼
                Decide
                   │
                   ▼
            DecisionResult
                   │
                   ▼
            Apply Decision
                   │
                   ▼
             Outside World
```

The transformations between the external boundaries form the pure evaluation core.

The external boundaries are responsible for interaction with the outside world.

---

## 4. Architectural Boundaries

DEO has two fundamental boundaries.

### 4.1 Input Boundary

The input boundary obtains the current state of the external energy system and translates it into domain concepts.

It is responsible for:

- obtaining current external state;
- obtaining forecasts and other external information;
- obtaining current time;
- translating infrastructure-specific representations; and
- constructing `SystemState`.

The input boundary does not make optimisation decisions.

---

### 4.2 Output Boundary

The output boundary receives `DecisionResult` and translates it into infrastructure-specific actions.

It is responsible for:

- interpreting the desired battery behaviour;
- comparing desired and current external state;
- making only required changes;
- applying inverter-specific operations; and
- returning the system to its normal operating mode when required for safety.

The output boundary does not reconsider the optimisation decision.

---

## 5. Evaluation Pipeline

The evaluation consists of four transformations.

```text
SystemState
    │
    ▼
 Validate
    │
    ▼
ValidatedState
    │
    ▼
  Assess
    │
    ▼
TrustedSystemState
    │
    ▼
  Decide
    │
    ▼
DecisionResult
```

Each transformation has one responsibility.

### 5.1 Validate

Establish that the supplied state is valid for the domain.

### 5.2 Assess

Establish which valid facts may be relied upon for the current evaluation.

### 5.3 Decide

Determine the correct charging behaviour from those trusted facts.

### 5.4 Apply Decision

Translate the decision into changes to the external system.

Apply Decision occurs outside the pure evaluation transformations.

---

## 6. Evaluation Architecture

The Deadline Energy Optimizer evaluates the current state of the world through a sequence of deterministic transformations over immutable data.

Each transformation has:

- a single responsibility;
- a defined immutable input contract;
- a defined immutable output contract; and
- no side effects.

The output of one transformation becomes the input to the next.

```text
                   Outside World
                         │
                         ▼
              Read Current World
                         │
                         ▼
              Construct SystemState
                         │
                         ▼
                  ┌──────────────┐
                  │   Validate   │
                  └──────┬───────┘
                         │
                         ▼
                 ValidatedState
                         │
                         ▼
                  ┌──────────────┐
                  │    Assess    │
                  └──────┬───────┘
                         │
                         ▼
               TrustedSystemState
                         │
                         ▼
                  ┌──────────────┐
                  │    Decide    │
                  └──────┬───────┘
                         │
                         ▼
                 DecisionResult
                         │
                         ▼
                 Apply Decision
                         │
                         ▼
                   Outside World
```

### 6.1 Construct SystemState

**Purpose**

Construct an immutable domain representation of the current world from information obtained through the input boundary.

**Responsibilities**

- Translate infrastructure-specific representations into domain concepts.
- Combine the current inputs required for an evaluation.
- Construct `SystemState`.

**Not responsible for**

- Determining whether values are valid.
- Determining whether information is trustworthy.
- Making optimisation decisions.
- Interacting with the inverter.

`SystemState` represents the world as it has been observed, not necessarily as it can safely be trusted.

### 6.2 Validate

**Purpose**

Establish that the supplied state and configuration satisfy the invariants required for further evaluation.

**Responsibilities**

- Validate configuration.
- Validate domain invariants.
- Reject malformed or internally inconsistent values.
- Produce `ValidatedState`.

Validation answers:

> **Are these inputs valid?**

It does not determine whether otherwise valid information is sufficiently trustworthy or available for optimisation.

### 6.3 Assess

**Purpose**

Establish the trusted facts available to the current evaluation.

**Responsibilities**

- Assess the availability and trustworthiness of information.
- Determine which facts may be used for optimisation.
- Determine whether degraded operation remains possible.
- Determine whether evaluation must terminate with a safe outcome.
- Produce `TrustedSystemState`.

Assessment answers:

> **Given these valid inputs, what can the optimiser safely rely upon?**

The reason a fact is unavailable is not relevant to later transformations. The assessment output represents the facts available for this evaluation.

### 6.4 Decide

**Purpose**

Determine the correct battery behaviour from the trusted facts and configuration.

**Responsibilities**

- Perform the calculations required to make the decision.
- Apply optimisation policy.
- Select the desired battery behaviour.
- Produce the derived facts materially required to explain that decision.
- Produce structured reason codes.
- Produce `DecisionResult`.

Decision answers:

> **Given these trusted facts, what should the battery do now?**

The decision is correct for the instant at which the world was evaluated. It is not a stored plan.

It remains the desired state until superseded by a subsequent evaluation.

### 6.5 Apply Decision

**Purpose**

Converge the external system toward the desired state represented by `DecisionResult`.

This activity occurs outside the pure evaluation transformations.

**Responsibilities**

- Interpret the implementation-independent decision.
- Compare the desired state with the current external state.
- Avoid changes when the desired state is already satisfied.
- Translate the decision into infrastructure-specific operations.
- Apply only the changes required.

Apply Decision answers:

> **What, if anything, must change in the outside world to satisfy this decision?**

The decision transformation therefore describes **what should happen**, while the output boundary determines **how to make it happen**.

---

## 7. Architectural Data Contracts

The evaluation architecture communicates exclusively through immutable data contracts.

Each contract represents a progressively more useful view of the current system.

```text
SystemState
     │
     ▼
ValidatedState
     │
     ▼
TrustedSystemState
     │
     ▼
DecisionResult
```

A contract does not expose how its values were produced. Consumers depend only on the guarantees of that contract.

### 7.1 SystemState

`SystemState` represents the current world as observed at the beginning of an evaluation.

It contains raw domain facts obtained from the architectural input boundary.

Examples include:

- current time;
- battery state of charge;
- battery capacity;
- current battery or inverter state;
- solar production;
- household consumption;
- forecast data;
- tariff information; and
- configuration.

`SystemState` shall contain observed facts only.

It shall not contain:

- optimisation calculations;
- inferred conclusions;
- trust decisions;
- fallback decisions; or
- derived values created for the purpose of optimisation.

Infrastructure-specific representations shall be translated into domain representations before inclusion in `SystemState`.

`SystemState` is immutable once constructed.

### Guarantee

`SystemState` answers:

> **What was observed?**

It makes no guarantee that those observations are valid or suitable for use in an optimisation decision.

---

### 7.2 ValidatedState

`ValidatedState` represents a `SystemState` whose supplied values satisfy the validity rules required by the domain.

Validation establishes that values are:

- present where required;
- represented using recognised types or enumerations;
- within permitted ranges;
- normalised into the expected domain representation; and
- mutually consistent with objective domain invariants.

Examples include:

- a state of charge within its valid range;
- a recognised operating mode;
- non-negative battery capacity;
- valid timestamps;
- recognised tariff structures; and
- internally consistent configuration.

Validation does not determine whether valid information is current, trustworthy or appropriate for use in a particular optimisation decision.

`ValidatedState` is immutable.

### Guarantee

A consumer of `ValidatedState` may assume:

> **Every supplied value is valid for the domain and internally consistent.**

---

### 7.3 TrustedSystemState

`TrustedSystemState` represents the facts that the Decision transformation may rely upon for the current evaluation.

Assessment begins with valid information and determines its usability.

A fact may be valid but unsuitable for use because, for example:

- a forecast is stale;
- forecast coverage does not extend far enough;
- historical data is insufficient to support an estimate;
- tariff information does not cover the required evaluation period; or
- a last-known external value is no longer considered current.

Assessment may therefore:

- carry a valid fact forward unchanged;
- make a fact unavailable to later transformations; or
- determine that insufficient trustworthy information exists to continue normal evaluation.

The Decision transformation shall not inspect the provenance of a fact or determine why it is unavailable.

`TrustedSystemState` is immutable.

### Guarantee

A consumer of `TrustedSystemState` may assume:

> **Every fact exposed by this contract may be relied upon for the current evaluation.**

---

### 7.4 DecisionResult

`DecisionResult` represents the answer to:

> **What do I need to do now for optimal charging?**

It describes the desired battery behaviour for the current evaluated state.

`DecisionResult` is not:

- a stored charging plan;
- a schedule for future execution;
- a record of previous decisions; or
- a vendor-specific hardware command.

The result remains authoritative until superseded by a subsequent evaluation.

In addition to the selected decision, `DecisionResult` shall contain only the derived information required to explain why that decision was made.

This may include:

- derived facts that materially influenced the decision; and
- structured reason codes.

It shall not duplicate the complete input state.

The exact structure and algorithm-specific contents of `DecisionResult` are defined in **040 — Core Algorithm**.

`DecisionResult` is immutable.

### Guarantee

A consumer of `DecisionResult` may assume:

> **This is the desired battery behaviour for the current evaluation, together with sufficient information to explain the decision.**

---

### 7.5 Contract Progression

Each contract adds a specific guarantee without changing the responsibility of earlier contracts.

```text
SystemState
"What was observed?"
        │
        ▼
ValidatedState
"Are the observations valid?"
        │
        ▼
TrustedSystemState
"What may Decision rely upon?"
        │
        ▼
DecisionResult
"What should happen now?"
```

No contract shall be mutated after creation.

Each transformation shall produce a new contract appropriate to its responsibility.

---

## 8. Purity Model

The Deadline Energy Optimizer separates its **pure evaluation core** from the **impure orchestration and external boundaries**.

The evaluation core performs deterministic transformations over immutable data and does not interact with the outside world.

The orchestrator coordinates an evaluation and is necessarily impure.

```text
                    Outside World
                         │
                         ▼
              ┌─────────────────────┐
              │    Orchestrator     │
              │      IMPURE         │
              │                     │
              │  Read Current World │
              └──────────┬──────────┘
                         │
                         ▼
                    SystemState
                         │
═════════════════════════╪═════════════════════════
                  PURE EVALUATION
                         │
                         ▼
                     Validate
                         │
                         ▼
                      Assess
                         │
                         ▼
                      Decide
                         │
                         ▼
                  DecisionResult
                         │
═════════════════════════╪═════════════════════════
                         │
                         ▼
              ┌─────────────────────┐
              │    Orchestrator     │
              │      IMPURE         │
              │                     │
              │   Apply Decision    │
              └──────────┬──────────┘
                         │
                         ▼
                    Outside World
```

### 8.1 Pure Evaluation Core

The evaluation core consists of transformations that operate exclusively on immutable input contracts.

Pure transformations:

- do not access external systems;
- do not read the system clock;
- do not perform network or filesystem operations;
- do not read or modify global state;
- do not depend on previous evaluations;
- do not mutate their inputs; and
- do not produce externally observable side effects.

Given identical inputs, the evaluation core shall produce an identical result.

Any information required by the evaluation, including the current time, shall be supplied through `SystemState`.

---

### 8.2 Impure Orchestration

The orchestrator coordinates one complete evaluation.

Its responsibilities include:

- obtaining the current state of the outside world;
- constructing `SystemState`;
- invoking the evaluation transformations in the required sequence;
- receiving `DecisionResult`;
- applying the decision through the output boundary; and
- handling unexpected failures at the evaluation boundary.

The orchestrator shall not contain validation, assessment or optimisation policy.

It coordinates those responsibilities but does not perform them.

---

### 8.3 External Boundaries

All interactions with external systems occur outside the pure evaluation core.

These interactions include:

- reading Home Assistant state;
- obtaining current time;
- obtaining forecasts;
- retrieving historical information;
- communicating with an inverter;
- applying hardware-specific commands;
- publishing externally visible state; and
- logging.

External representations shall be translated into domain contracts before entering the pure evaluation core.

`DecisionResult` shall be translated into infrastructure-specific operations only after leaving the pure evaluation core.

---

### 8.4 Purity Boundary

Purity is a property of the **evaluation core**, not of DEO as a whole.

DEO necessarily interacts with the outside world in order to observe the system and apply decisions.

The architecture isolates those interactions so that the question:

> **What do I need to do now for optimal charging?**

can be answered without interacting with the outside world.

---

## 9. Failure Model

The Deadline Energy Optimizer distinguishes between **expected operational conditions** and **unexpected failures**.

Expected conditions are part of normal evaluation and are represented explicitly in the domain.

Unexpected failures indicate that the evaluation cannot be trusted and shall cause the current evaluation to fail fast.

Safety takes precedence over continuing optimisation.

```text
                         Evaluation
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
       Expected Condition        Unexpected Failure
                │                         │
                ▼                         ▼
       Represented as Data            Fail Fast
                │                         │
                ▼                         ▼
       Evaluate Deliberately     Restore Safe State
                │                         │
                ▼                         ▼
         DecisionResult             End Evaluation
```

### 9.1 Expected Operational Conditions

Expected operational conditions are situations that the optimiser is designed to encounter during normal operation.

Examples include:

- stale or unavailable forecast data;
- insufficient historical information;
- incomplete forecast coverage;
- unavailable optional inputs;
- an infeasible optimisation target;
- degraded external capability; and
- invalid input or configuration.

These conditions shall not be represented as unexpected programming exceptions.

They shall be detected by the transformation responsible for understanding them and represented explicitly in its output contract.

Depending on the condition, evaluation may:

- continue normally;
- continue using a defined fallback;
- produce a conservative decision; or
- terminate with a defined safe decision.

The exact behaviour for each expected condition is defined by **040 — Core Algorithm**.

---

### 9.2 Unexpected Failures

An unexpected failure is a condition outside the expected domain behaviour that prevents the correctness of the current evaluation from being guaranteed.

Examples include programming defects and violated assumptions that should have been prevented by the architecture or implementation.

Unexpected failures shall fail fast.

Individual evaluation transformations shall not convert unexpected programming defects into ordinary domain outcomes merely to allow evaluation to continue.

Continuing after an unexpected failure could produce an apparently valid but incorrect decision.

The current evaluation shall therefore terminate.

---

### 9.3 Safe-State Restoration

When an unexpected failure occurs, the orchestrator shall attempt to return the energy system to its defined safe state.

The safe state is:

> **The normal operating mode of the energy system outside DEO optimisation control.**

Safe-state restoration shall:

- relinquish active DEO optimisation control;
- remove DEO-requested forced charging or equivalent control;
- return the inverter to its normal autonomous operating behaviour;
- avoid leaving equipment in a potentially unsafe operating state; and
- avoid leaving an uncontrolled mode that could cause unnecessarily expensive energy consumption.

The safe state is not required to be optimal.

Its purpose is to leave the physical system in a known, independently safe operating mode when DEO can no longer guarantee correct optimisation.

The infrastructure-specific definition of normal operating mode belongs to the output adapter.

---

### 9.4 Safe-State Restoration Failure

Restoring the safe state requires interaction with the outside world and may itself fail.

The output boundary shall make a bounded attempt to restore the safe state.

The number, timing and implementation of retries are not architectural concerns and are defined by the implementation.

If safe-state restoration cannot be confirmed after the bounded attempt, DEO shall:

- record the original evaluation failure;
- record the safe-state restoration failure;
- raise a prominent operational notification;
- report the resulting external state as unknown; and
- terminate the current evaluation.

DEO shall never report that the system has been returned to a safe state unless that outcome has been confirmed.

Once communication with the controlled system has failed, DEO cannot guarantee its physical state.

---

### 9.5 Recovery

DEO does not maintain a recovery state machine.

Each subsequent evaluation begins independently from the newly observed state of the world.

```text
Evaluation N
     │
     ▼
Unexpected Failure
     │
     ▼
Safe-State Attempt
     │
     ▼
End Evaluation


        time


Evaluation N + 1
     │
     ▼
Read Current World
     │
     ▼
New Independent Evaluation
```

No failed evaluation is resumed.

No previous optimisation decision is assumed to remain correct.

Recovery therefore follows directly from the stateless architecture:

> **Observe the world again and make a new decision.**

This keeps failure handling consistent with the same execution model used during normal operation.

---

## 10. Traceability

The architecture exists to satisfy the requirements defined in **020 — Requirements**.

Traceability closes the loop between requirements and architectural responsibilities:

```text
Requirements
     │
     │  Why must the system do this?
     ▼
Architecture
     │
     │  Where is that responsibility satisfied?
     ▼
Core Algorithm
     │
     │  What behaviour implements it?
     ▼
Tests
     │
     ▼
Verified Behaviour
```

Architectural traceability shall be maintained only where a requirement has an architectural consequence.

A lightweight mapping shall identify:

| Requirement | Architectural Responsibility |
|---|---|
| Safety requirements | Assessment, Failure Model, Safe-State Restoration |
| Input validity requirements | Validate, `ValidatedState` |
| Input trust and availability requirements | Assess, `TrustedSystemState` |
| Optimisation requirements | Decide, `DecisionResult` |
| Stateless operation | Stateless Evaluation, Orchestration |
| Explainability | `DecisionResult` |
| External system interaction | Input and Output Boundaries |
| Deterministic behaviour | Pure Evaluation Core, Immutable Contracts |

Detailed behavioural traceability belongs in **040 — Core Algorithm** and its tests.

The architecture shall not duplicate requirement definitions. **020 — Requirements remains authoritative for what the system is required to do.**
