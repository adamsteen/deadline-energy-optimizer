# Deadline Energy Optimizer

## Project Charter

| Field | Value |
|---|---|
| Project Name | Deadline Energy Optimizer |
| Document | Project Charter |
| Specification Version | 0.1.0 |
| Document Version | 0.1 |
| Status | DRAFT |
| Last Updated | 2026-07-31 |

## Abstract

Deadline Energy Optimizer (DEO) is a specification-first Home Assistant project that determines the lowest-cost battery charging strategy required to achieve a configured target state of charge before a configured deadline.

## Purpose

The project exists to make household battery charging decisions that are safe, understandable, deterministic and economical. It should use the cheapest available energy while still ensuring that the battery reaches the required state of charge by the required time.

## Objectives

DEO shall:

- use forecast solar generation when planning battery charging;
- account for expected household consumption;
- prefer solar energy over grid energy when doing so does not endanger the deadline;
- use cheap grid energy when required;
- increase charging power when remaining time is insufficient at the current rate;
- continuously revise the charging plan as conditions and forecasts change;
- fail safely and restore the inverter to a known safe state when disabled or when an error occurs.

## Scope

The initial project scope includes:

- a pure decision engine;
- forecast, household-load and inverter adapters;
- Home Assistant orchestration;
- deterministic configuration with sane defaults;
- automated unit and integration tests;
- operational logging and diagnostics.

The initial project does not include:

- general-purpose energy trading;
- export-price optimisation;
- support for every inverter vendor;
- machine-learning-based decisions where simpler deterministic logic is sufficient.

## Delivery Lifecycle

The project follows this lifecycle:

> Idea → Discussion → Draft Specification → Review → LOCKED → Architecture Review → Implementation → Unit Tests → Integration Tests → Release

Implementation must not begin until the relevant specification is LOCKED.

## Engineering Principles

1. Safety First
2. KISS
3. YAGNI
4. Design for Understanding
5. Deterministic Behaviour
6. Principle of Least Astonishment

Supporting principles are:

- Documentation First
- Specification as Source of Truth
- Separation of Concerns
- Single Responsibility
- Testability
- DRY for knowledge
- A regression test for every bug

When principles conflict, prefer the option that is, in order:

1. safest;
2. simplest;
3. most understandable;
4. most deterministic;
5. most maintainable.

## Governance

Specifications use the statuses DRAFT, REVIEW, LOCKED and SUPERSEDED. Decisions are recorded in the specification set; discussion history is not a substitute for a decision.

> Don't document discussions; document decisions.

## Revision History

| Version | Date | Status | Description |
|---|---|---|---|
| 0.1 | 2026-07-31 | DRAFT | Initial project charter |

## Assumptions

- Home Assistant is the initial orchestration environment.
- A supported inverter can expose battery state and accept charging controls.
- At least one solar forecast source is available.
- The system has a tariff period in which grid charging may be economically useful.

## Open Issues

- Confirm the initial inverter adapter and exact supported controls.
- Confirm the initial forecast provider and update cadence.
- Confirm licensing before the first release.

## References

- [000 — Introduction](000-introduction.md)
- [010 — Problem Statement](010-problem-statement.md)
- [020 — Requirements](020-requirements.md)
- [030 — Architecture](030-architecture.md)
