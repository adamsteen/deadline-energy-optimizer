# Deadline Energy Optimizer

# 010 — Problem Statement

| Field | Value |
|---|---|
| Project Name | Deadline Energy Optimizer |
| Document Title/Number | 010 — Problem Statement |
| Specification Version | 0.1.0 |
| Document Version | 0.1 |
| Status | DRAFT |
| Last Updated | 2026-07-31 |

## Abstract

Household batteries can be charged from solar generation or from the electricity grid. The cheapest charging strategy depends on battery state, expected household consumption, electricity tariffs, available charging power, the remaining time before a deadline, and uncertain solar generation later in the day. A fixed schedule cannot reliably account for these changing conditions.

Deadline Energy Optimizer addresses this problem by continuously determining whether charging should be deferred, started, continued, slowed or accelerated so that a configured target state of charge is achieved by a configured deadline at the lowest practical cost.

## 1. Context

A household with rooftop solar, a battery and time-of-use electricity pricing can often charge the battery from either:

- surplus solar generation;
- low-cost grid electricity; or
- a combination of both.

The economically correct choice varies from day to day. For example:

- a cloudy morning may be followed by a sunny afternoon, making early grid charging unnecessary;
- a sunny morning may fill the battery early, followed by poor afternoon generation and high household demand;
- a forecast may deteriorate, making grid charging necessary later in the charging window;
- insufficient time may remain to reach the target at a low charging rate, requiring faster charging.

## 2. Problem

Existing fixed schedules and simple threshold automations do not adequately solve the charging problem because they generally do not combine all relevant information or revise their decisions as conditions change.

A useful optimiser must answer, repeatedly:

> What charging action is required now to reach the target battery state of charge by the deadline while using the lowest-cost energy that can reasonably be expected to be available?

## 3. Inputs Affecting the Decision

The decision is affected by:

- current battery state of charge;
- target battery state of charge;
- battery capacity and usable capacity;
- configured deadline;
- time remaining before the deadline;
- available normal and fast charging rates;
- forecast solar generation before the deadline;
- expected household consumption before the deadline;
- current and future electricity prices;
- inverter and battery operating limits;
- forecast age, availability and uncertainty;
- previous optimiser actions and current inverter state.

## 4. Why the Problem Is Non-Trivial

### 4.1 Solar energy is uncertain

Solar forecasts are estimates and may change during the day. Treating forecast production as guaranteed can leave the battery below target.

### 4.2 Household consumption competes with charging

Forecast solar production is not automatically available to the battery. Household consumption must be supplied first, so only expected surplus generation can contribute to charging.

### 4.3 Delaying grid charging has value and risk

Waiting for forecast solar may reduce cost, but excessive delay may leave insufficient time to charge from the grid before the deadline.

### 4.4 Charging power is constrained

The battery and inverter impose maximum charging rates. The optimiser must recognise when the remaining energy requirement cannot be delivered in the remaining time at the current rate.

### 4.5 Conditions change continuously

Battery state, household load, solar production and forecasts evolve throughout the charging period. A plan that was correct earlier may no longer be correct.

### 4.6 Safety overrides optimisation

Cost reduction is secondary to safe operation. Invalid inputs, communication failures, disabled operation or runtime errors must not leave the inverter in an unsafe or unexpected state.

## 5. Desired Outcome

The optimiser should make conservative, deterministic and explainable decisions that:

1. achieve the configured target state of charge by the configured deadline whenever physically possible;
2. minimise the cost of energy used to achieve that target;
3. prefer forecast surplus solar over grid charging when doing so does not place the deadline at unacceptable risk;
4. use low-cost grid energy when forecast solar is insufficient or too uncertain;
5. increase charging power when the remaining time requires it;
6. revise the plan as new measurements and forecasts become available;
7. respect inverter, battery and configured safety limits;
8. return the inverter to a known safe state when the optimiser is disabled or cannot operate reliably.

## 6. Scope Boundary

This problem statement concerns charging the battery to a target by a deadline. It does not define:

- the detailed decision algorithm;
- Home Assistant entity names;
- vendor-specific inverter commands;
- user-interface design;
- export or electricity-market trading optimisation;
- general optimisation of all household loads.

Those concerns belong in later specifications or adapter implementations.

## 7. Success Criteria

The problem is considered solved when the system can demonstrate, through deterministic tests and operational diagnostics, that it:

- reaches the target by the deadline when feasible;
- identifies when the target is physically infeasible;
- avoids unnecessary grid charging when sufficient solar surplus is reasonably expected;
- begins grid charging early enough when solar is insufficient;
- selects faster charging when required by the remaining time;
- responds predictably to changing forecasts and measurements;
- fails safely.

## Revision History

| Version | Date | Status | Description |
|---|---|---|---|
| 0.1 | 2026-07-31 | DRAFT | Initial problem statement |

## Assumptions

- Battery charging can be controlled through an inverter adapter.
- Battery state of charge and relevant power measurements are available.
- At least one solar forecast provider is available.
- Electricity pricing can be represented for the optimisation period.
- The deadline and target state of charge are configurable.

## Open Issues

- Define how forecast uncertainty is represented.
- Define the minimum reserve or safety margin applied to predicted solar surplus.
- Define behaviour when the target is physically infeasible.
- Define whether multiple tariff changes before the deadline are supported initially.

## References

- [Project Charter](project-charter.md)
- [000 — Introduction](000-introduction.md)
- [020 — Requirements](020-requirements.md)
- [030 — Architecture](030-architecture.md)
- [040 — Core Algorithm](040-core-algorithm.md)
