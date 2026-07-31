# Deadline Energy Optimizer

# 030 — Architecture

| Field | Value |
|---|---|
| Specification Version | 0.1.0 |
| Document Version | 0.1 |
| Status | DRAFT |

## Abstract

The Deadline Energy Optimizer architecture is a layered architecture centred around a pure decision engine. External systems provide inputs through adapters and execute optimiser decisions through adapters. The core decision engine remains independent of Home Assistant, inverter vendors and forecast providers.

## Architectural Principles

- Pure, deterministic decision engine.
- Adapter pattern for all external integrations.
- Separation of concerns.
- Dependency inversion.
- Test-first design.

## Layers

1. Home Assistant orchestration.
2. Provider adapters (forecast, load, inverter).
3. Core decision engine.
4. Domain model and pure calculations.

## Data Flow

Providers → Adapters → Validation → Decision Engine → Decision → Inverter Adapter → Hardware

## Failure Boundaries

Adapter failures shall not corrupt decision logic. Invalid or stale inputs result in safe behaviour as defined by the requirements specification.

## Deployment

The initial implementation targets Home Assistant using Pyscript while keeping the core engine portable.

## References

- 020 — Requirements
- 040 — Core Algorithm
