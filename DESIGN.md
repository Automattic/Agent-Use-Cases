# Agent-Use-Cases — Design

> A Claude Code plugin marketplace of WordPress agent use cases

This document is the source of truth for the current architecture and behavior of Agent-Use-Cases. If something in the code contradicts this document, this document wins. If this document is wrong, update it — don't silently diverge. Keep it updated whenever adding, changing, or removing features. Use `.agents/decisions/` for durable rationale about why major choices were made.

## Overview

<!-- What is this system? Who uses it? What problem does it solve? -->

## Goals

<!-- What does success look like? -->

## Non-goals

<!-- What is explicitly out of scope? -->

## Architecture

<!-- Components, data flow, how the system fits together. Add diagrams as needed. -->

## Invariants

<!-- Properties that MUST always hold across the system, distinct from current-state architecture
prose above. Architecture describes structure ("here's what the components are"); Invariants
describe rules that constrain how the system can change ("regardless of how we restructure, X
must always be true"). Examples:

- Security boundary: requests from untrusted origins must pass through the auth middleware before
  reaching any handler that touches user data.
- Data flow: any write to the orders table must emit an event on the order-events stream within
  the same transaction.
- Architectural rule: domain logic does not import from infrastructure layers; only the inverse
  is allowed.

When you change code that violates a stated invariant, either (a) update the code to preserve
it, or (b) explicitly remove/revise the invariant in the same commit, with a body explaining
what changed and why. -->

## Decision index

<!-- Link durable decision records from .agents/decisions/ when they explain current architecture,
invariants, public behavior, persistence shape, security posture, or other choices future agents are
likely to question. DESIGN.md states what is true now; decision records explain why. -->

## Open questions

<!-- Unresolved design questions. Move to FOLLOW_UPS.md or IDEAS.md as they crystallize. -->
