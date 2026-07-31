# Deadline Energy Optimizer

# 000 — Introduction

| Field | Value |
|---|---|
| Specification Version | 0.1.0 |
| Document Version | 0.1 |
| Status | DRAFT |
| Last Updated | 2026-07-31 |

## Abstract

Deadline Energy Optimizer (DEO) is a Home Assistant application that plans battery charging so a configured state of charge is achieved by a configured deadline using the lowest-cost mix of solar and grid energy.

## Goals

- Meet the battery SOC deadline.
- Minimise charging cost.
- Prefer solar where practical.
- Use deterministic, explainable decisions.
- Operate safely under all conditions.

## Philosophy

DEO is specification-first. Requirements define what the system must do, architecture defines how components interact, and algorithms define how decisions are made. Implementation follows only after specifications are locked.

## Document Structure

- 010 — Problem Statement
- 020 — Requirements
- 030 — Architecture
- 040 — Core Algorithm

## Assumptions

The optimiser executes within Home Assistant but keeps its decision engine independent of Home Assistant-specific APIs wherever practical.

## Open Issues

None.
