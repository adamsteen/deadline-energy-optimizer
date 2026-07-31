# Deadline Energy Optimizer

# 020 — Requirements

| Field | Value |
|---|---|
| Project Name | Deadline Energy Optimizer |
| Document Title/Number | 020 — Requirements |
| Specification Version | 1.0.0 |
| Document Version | 1.0 |
| Status | LOCKED |
| Last Updated | 2026-07-31 |

## Abstract

This specification defines what Deadline Energy Optimizer (DEO) must do. It establishes the functional, safety, performance, configuration, operational, diagnostic, testing and non-functional requirements against which the architecture, algorithm and implementation shall be assessed.

This document intentionally does not prescribe implementation details.

## 1. Purpose

DEO shall determine and apply the charging action required to reach a configured battery state of charge by a configured deadline while minimising charging cost and preserving safe, predictable operation.

## 2. Scope

This specification covers:

- battery charging decisions;
- solar forecast use;
- household consumption estimation;
- tariff-aware charging;
- normal and fast charging decisions;
- repeated replanning;
- safe disable and failure behaviour;
- configuration, observability and testability.

This specification does not define:

- vendor-specific inverter protocols;
- Home Assistant entity names;
- detailed algorithm formulas;
- user-interface layout;
- export-market optimisation;
- unrelated household load scheduling.

## 3. Requirement Language

The words **shall**, **must**, **should** and **may** are used as follows:

- **shall** and **must** indicate mandatory requirements;
- **should** indicates a preferred behaviour that may be departed from with justification;
- **may** indicates optional behaviour.

## 4. Functional Requirements

### REQ-F-001 — Target and deadline

The optimiser shall accept a configurable target state of charge and a configurable deadline.

### REQ-F-002 — Current battery state

The optimiser shall use the current battery state of charge when determining the remaining charging requirement.

### REQ-F-003 — Required energy

The optimiser shall determine the energy required to move the battery from its current state of charge to the configured target, subject to configured usable battery capacity and operating limits.

### REQ-F-004 — Deadline achievement

The optimiser shall plan charging so that the target state of charge is reached by the deadline whenever this is physically possible.

### REQ-F-005 — Infeasibility detection

The optimiser shall detect when the target cannot be reached by the deadline within the configured battery and inverter limits.

### REQ-F-006 — Cost minimisation

Subject to safety and deadline achievement, the optimiser shall minimise the cost of energy used to reach the target.

### REQ-F-007 — Solar forecast

The optimiser shall use forecast solar generation that is expected before the deadline.

### REQ-F-008 — Household consumption

The optimiser shall account for expected household consumption before treating forecast solar generation as available for battery charging.

### REQ-F-009 — Forecast surplus

The optimiser shall distinguish forecast solar generation from forecast solar surplus available to charge the battery.

### REQ-F-010 — Solar preference

The optimiser shall prefer expected solar surplus over grid charging when doing so does not create an unacceptable risk of missing the deadline.

### REQ-F-011 — Grid charging

The optimiser shall use grid charging when forecast solar surplus is insufficient, unavailable, stale or too uncertain to meet the target reliably.

### REQ-F-012 — Tariff awareness

The optimiser shall consider the applicable electricity price during the optimisation period when selecting grid charging actions.

### REQ-F-013 — Deferred charging

The optimiser shall be able to defer grid charging when sufficient lower-cost energy is reasonably expected to become available before the deadline.

### REQ-F-014 — Normal charging

The optimiser shall support a configured normal charging rate.

### REQ-F-015 — Fast charging

The optimiser shall support a configured fast charging rate where the inverter and battery permit it.

### REQ-F-016 — Fast-charge necessity

The optimiser shall enable fast charging only when necessary to protect the deadline or when explicitly required by a locked future requirement.

### REQ-F-017 — Remaining-time assessment

The optimiser shall compare the remaining charging requirement with the energy that can be delivered during the remaining time at available charging rates.

### REQ-F-018 — Continuous replanning

The optimiser shall recalculate its charging plan at a configurable interval.

### REQ-F-019 — Event-driven replanning

The optimiser should recalculate when a materially relevant input changes, provided this does not violate rate limits or stability constraints.

### REQ-F-020 — Revised forecasts

The optimiser shall revise its charging plan when updated forecasts materially change the expected energy available before the deadline.

### REQ-F-021 — Revised measurements

The optimiser shall revise its charging plan as battery state, solar production, household consumption or inverter state changes.

### REQ-F-022 — Enabled state

The optimiser shall expose a controllable enabled/disabled state suitable for use by Home Assistant automations and dashboards.

### REQ-F-023 — No-action decision

The optimiser shall be able to determine that no charging action is currently required.

### REQ-F-024 — Explicit decision output

Each evaluation shall produce an explicit decision describing the required charging mode and the reason for that decision.

### REQ-F-025 — Deadline completion

When the target state of charge has been reached, the optimiser shall stop requesting deadline-driven grid charging unless another locked requirement applies.

## 5. Safety Requirements

### REQ-S-001 — Safety priority

Safety shall take precedence over cost optimisation and deadline achievement.

### REQ-S-002 — Hardware limits

The optimiser shall respect configured and reported battery and inverter charging limits.

### REQ-S-003 — State-of-charge bounds

The optimiser shall not request operation outside configured battery state-of-charge bounds.

### REQ-S-004 — Charging-rate bounds

The optimiser shall not request a charging rate greater than the lowest applicable configured, battery-reported or inverter-reported limit.

### REQ-S-005 — Invalid input handling

The optimiser shall not issue a new charging command based on invalid, unavailable or internally inconsistent mandatory inputs.

### REQ-S-006 — Stale input handling

The optimiser shall detect mandatory inputs that exceed their configured maximum age and shall apply a documented safe behaviour.

### REQ-S-007 — Disable behaviour

When disabled, the optimiser shall stop making optimisation decisions and shall restore the inverter to a configured safe state.

### REQ-S-008 — Error behaviour

On an unhandled evaluation or execution error, the optimiser shall restore or request restoration of the inverter to a configured safe state.

### REQ-S-009 — Startup behaviour

At startup, the optimiser shall not assume that the inverter is already in the desired state.

### REQ-S-010 — Command verification

The system shall provide a means to detect whether an requested inverter state was not applied or was subsequently lost.

### REQ-S-011 — Command idempotence

Repeated application of the same optimiser decision shall not create unsafe or materially different behaviour.

### REQ-S-012 — Fail-safe configuration

Missing mandatory configuration shall prevent active optimisation rather than cause implicit unsafe defaults.

### REQ-S-013 — Safe defaults

Optional configuration shall have documented, conservative defaults.

### REQ-S-014 — Manual control

Disabling the optimiser shall allow manual or external control to resume without the optimiser immediately reasserting its previous command.

## 6. Performance Requirements

### REQ-P-001 — Evaluation duration

A normal optimiser evaluation shall complete quickly enough for routine Home Assistant automation use and shall not block unrelated Home Assistant processing.

### REQ-P-002 — Forecast API limits

The system shall support forecast refresh scheduling that respects provider API limits.

### REQ-P-003 — Local evaluation

Routine decision evaluations shall not require a new external forecast API call when a sufficiently recent forecast is already available locally.

### REQ-P-004 — Command minimisation

The optimiser should avoid sending redundant inverter commands when the requested state is already active.

### REQ-P-005 — Stable decisions

The optimiser shall include sufficient stability controls to avoid rapid oscillation between charging modes in response to insignificant input changes.

### REQ-P-006 — Deterministic execution

For the same validated inputs and configuration, the decision engine shall return the same decision.

## 7. Configuration Requirements

### REQ-C-001 — YAML configuration

The complete initial optimiser configuration shall be expressible in YAML.

### REQ-C-002 — Sane defaults

Optional settings shall have documented sane defaults suitable for a typical supported installation.

### REQ-C-003 — Required configuration

The system shall clearly distinguish mandatory configuration from optional configuration.

### REQ-C-004 — Validation

Configuration shall be validated before active optimisation begins.

### REQ-C-005 — Target configuration

Target state of charge shall be configurable.

### REQ-C-006 — Deadline configuration

The deadline shall be configurable.

### REQ-C-007 — Battery capacity

Usable battery capacity shall be configurable or obtainable from a validated adapter input.

### REQ-C-008 — Charging rates

Normal and fast charging rates shall be configurable or obtainable from validated adapter inputs.

### REQ-C-009 — Recalculation interval

The normal recalculation interval shall be configurable.

### REQ-C-010 — Input freshness

Maximum acceptable ages for time-sensitive inputs shall be configurable.

### REQ-C-011 — Safety margins

Forecast and charging safety margins shall be configurable where applicable.

### REQ-C-012 — Safe state

The inverter state required when the optimiser is disabled or fails shall be configurable.

### REQ-C-013 — Provider selection

Forecast, load and inverter providers shall be selectable without changing the core decision engine.

## 8. Operational Requirements

### REQ-O-001 — Home Assistant operation

The initial release shall operate within or alongside Home Assistant.

### REQ-O-002 — User-visible status

The system shall expose the optimiser's current enabled state, decision and operating status to Home Assistant.

### REQ-O-003 — Decision reason

The system shall expose a concise human-readable reason for the current decision.

### REQ-O-004 — Required energy status

The system shall expose the currently calculated energy required to reach the target.

### REQ-O-005 — Forecast contribution status

The system shall expose the forecast contribution expected to be available for charging before the deadline.

### REQ-O-006 — Grid requirement status

The system shall expose the currently calculated grid-energy requirement.

### REQ-O-007 — Feasibility status

The system shall expose whether the target is currently expected to be feasible.

### REQ-O-008 — Time remaining

The system shall expose the time remaining before the deadline.

### REQ-O-009 — Restart recovery

After restart, the system shall re-evaluate current inputs and reconcile the inverter state rather than relying on transient in-memory state.

### REQ-O-010 — Provider independence

Replacing one supported forecast or inverter provider shall not require changes to the core decision rules.

## 9. Logging and Diagnostics Requirements

### REQ-L-001 — Evaluation logging

The system shall log each materially different decision and the reason for it.

### REQ-L-002 — Input summary

Diagnostic output shall provide the validated inputs used for a decision, subject to appropriate redaction and verbosity controls.

### REQ-L-003 — Calculation summary

Diagnostic output shall provide the key intermediate quantities needed to understand a decision.

### REQ-L-004 — Command logging

The system shall log inverter commands requested by the optimiser and their outcomes.

### REQ-L-005 — Failure logging

The system shall log invalid inputs, stale inputs, adapter failures, rejected commands and safe-state actions.

### REQ-L-006 — No secret leakage

Logs shall not expose API keys, credentials or other secrets.

### REQ-L-007 — Explainability

An operator shall be able to determine from exposed status and logs why grid charging was started, deferred, accelerated or stopped.

## 10. Testing Requirements

### REQ-T-001 — Pure-function tests

Core calculations and decision rules shall be implemented so they can be tested as pure functions wherever practical.

### REQ-T-002 — Unit tests

Each decision rule and material calculation shall have automated unit tests.

### REQ-T-003 — Algorithm scenarios

The test suite shall include representative scenarios for sunny, cloudy, changing-forecast and insufficient-time conditions.

### REQ-T-004 — Boundary tests

The test suite shall cover state-of-charge, deadline, charging-rate, tariff-boundary and forecast-freshness boundaries.

### REQ-T-005 — Safety tests

The test suite shall verify disable, invalid-input, stale-input, command-failure and exception safe-state behaviour.

### REQ-T-006 — Infeasibility tests

The test suite shall verify detection and reporting of physically infeasible targets.

### REQ-T-007 — Determinism tests

The test suite shall verify that identical validated inputs produce identical decisions.

### REQ-T-008 — Adapter contract tests

Each adapter shall have tests demonstrating conformance with its defined input or output contract.

### REQ-T-009 — Integration tests

The project shall include integration tests covering orchestration between the decision engine and representative adapters.

### REQ-T-010 — Regression tests

Every confirmed defect shall receive an automated regression test where technically practical.

### REQ-T-011 — Manual validation

The project shall document a safe procedure for manually validating charging behaviour on a real installation.

## 11. Non-Functional Requirements

### REQ-N-001 — Separation of concerns

The core decision engine shall remain independent of Home Assistant, inverter vendors and forecast-provider APIs.

### REQ-N-002 — Understandability

The design and implementation shall favour code that can be understood and reviewed over unnecessarily compact or clever solutions.

### REQ-N-003 — Maintainability

Configuration, adapters, decision logic and orchestration shall be separated so that changes in one area have minimal impact on others.

### REQ-N-004 — Extensibility

The architecture shall permit additional forecast and inverter adapters without changing locked decision requirements.

### REQ-N-005 — Simplicity

The implementation shall not introduce complexity without a current requirement that justifies it.

### REQ-N-006 — Documentation

Public interfaces, configuration and operational behaviour shall be documented.

### REQ-N-007 — Traceability

Architecture, implementation and tests shall reference applicable requirement identifiers.

### REQ-N-008 — Version control

Specifications and source code shall be maintained in version control with meaningful commits.

### REQ-N-009 — Dependency control

External dependencies shall be limited to those that provide clear value and shall be documented.

### REQ-N-010 — Time handling

Deadline calculations shall use explicit, timezone-aware times and shall behave predictably across date boundaries.

## 12. Acceptance Conditions

An implementation may claim conformance with this specification only when:

- every mandatory requirement is implemented or explicitly marked not applicable with justification;
- automated tests demonstrate the required decision and safety behaviour;
- the implementation can explain its charging decisions operationally;
- relevant architecture and algorithm specifications are LOCKED;
- known safety-critical defects are resolved.

## Revision History

| Version | Date | Status | Description |
|---|---|---|---|
| 1.0 | 2026-07-31 | LOCKED | Initial approved requirements baseline |

## Assumptions

- Home Assistant is the initial orchestration environment.
- The inverter exposes sufficient monitoring and control through an adapter.
- Solar forecast data is available locally or through a provider integration.
- Household consumption can be estimated from measurements or historical data.
- Electricity prices applicable before the deadline are known or configured.

## Open Issues

The following issues are intentionally deferred to later specifications and do not change the locked requirements baseline:

- the exact representation of forecast uncertainty;
- the calculation of forecast safety margins;
- the precise fast-charge threshold;
- the initial forecast-provider adapter;
- the initial inverter-provider adapter;
- the detailed user-interface presentation.

## References

- [Project Charter](project-charter.md)
- [000 — Introduction](000-introduction.md)
- [010 — Problem Statement](010-problem-statement.md)
- [030 — Architecture](030-architecture.md)
- [040 — Core Algorithm](040-core-algorithm.md)
