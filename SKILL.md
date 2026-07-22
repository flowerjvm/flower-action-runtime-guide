---
name: flower-action-runtime-guide
description: Use when building, modifying, reviewing, migrating, or testing Java code that uses flower-action-runtime, including ActionProposal and ActionDefinition design, registry/validation/policy/approval/pre-execution controls, tenant-scoped duplicate handling, explicit failure retry policy, synchronous/async/deferred executor selection, ActionRun and RunStore persistence, JDBC CAS concurrency, completion/cancellation/recovery, Flower workflow or event-loop backends, and 0.1/0.2-to-0.3 migration.
---

# Flower Action Runtime Guide

## Overview

Use this skill to keep business actions behind one explicit control boundary
while supporting approval, audit, asynchronous work, durable completion, and
safe concurrent state changes.

This skill owns action-runtime guidance. When the host also authors Flower
Flows, Steps, Guards, Workers, event waits, or checkpoints, also load the
sibling `flower-app-guide` skill and follow both sets of rules.

## Start Here

Always read:

- `references/00-guide-version.md`
- `references/01-runtime-quick-rules.md`
- `references/90-verification.md`

Then read the reference that matches the task.

## Reference Routing

- Designing actions, proposal identity, definitions, policy, approval,
  pre-execution checks, results, audit, or idempotency: read
  `references/10-action-model-and-controls.md`.
- Choosing synchronous, in-process async, or durable deferred execution and
  implementing completion/cancellation: read
  `references/20-execution-modes.md`.
- Working with `ActionRun`, custom `RunStore`, JDBC schema/migrations, CAS,
  recovery, or multiple server instances: read
  `references/30-persistence-and-concurrency.md`.
- Selecting `DefaultActionRuntime`, the workflow backend, or the event-loop
  backend: read `references/40-flower-backends.md`.
- Writing unit, parity, concurrency, persistence, or recovery tests: read
  `references/50-testing.md`.
- Integrating the runtime into REST, MCP, schedulers, AI planners, Spring, or a
  domain application: read `references/60-host-integration.md`.

## Workflow

1. Identify the business side effect and every entry point that can request it.
2. Inspect the runtime version and public APIs in the checked-out source before
   writing code; do not assume the Maven Central API matches the checked-out
   pre-1.0 source line.
3. Define a stable action id and register exactly one controlled executor.
4. Map request channel, proposer type, execution principal, tenant, run id, and
   idempotency key without collapsing them into one actor field. In 0.3,
   `ActionOrigin` no longer exists.
5. Configure validation, policy, approval, duplicate handling, audit/trace, and
   the pre-execution guard before exposing the action.
6. Choose the executor mode by lifetime and durability, not by convenience.
7. Use a queryable durable `RunStore` for approval, async, deferred, callback,
   cancellation, or restart-sensitive work.
8. If Flower drives the backend, keep Worker/EventWorker ticks non-blocking and
   keep governance semantics in the shared action pipeline.
9. Add focused failure, resume, race, and recovery tests, then run the complete
   verification appropriate to the changed modules.

## Non-Negotiable Rules

- Treat UI, REST, batch, MCP, scheduler, and AI output as proposals. Only a
  registered `ActionExecutor` may perform the controlled side effect.
- Never bypass registry, validation, policy, approval, duplicate handling, or
  audit in a resume, callback, retry, recovery, or admin path.
- Re-resolve, re-validate, re-evaluate policy, and run `PreExecutionGuard`
  after approval and immediately before dispatch.
- Do not block Flower Worker or EventWorker ticks with domain work, HTTP, LLM,
  tools, sleeps, or `Future.get()`.
- Use `ActionExecutionResult.code()` and `RetryDisposition` for machine
  decisions. Do not parse human messages or leave failure retry safety
  implicit.
- Give every action request a unique `runId`. Use the idempotency key to group
  transport retries; do not reuse a run id for a new request.
- Make every `RunStore` transition versioned CAS. Do not add an unconditional
  update/upsert path to the runtime SPI.
- Authenticate callback callers and verify tenant, run id, and attempt token.
- Make external cancellation hooks idempotent for the same run attempt and
  operation id.
- Scope duplicate reservations by at least tenant, action id, and idempotency
  key. Add principal or resource scope when an existing result is not safely
  shareable within the tenant.
- Resolve, validate, and authorize the current request before duplicate
  reservation or `RETURN_EXISTING` result lookup.
- Treat `CANCELLED` as the runtime's terminal acceptance decision, not proof
  that an external operation physically stopped.
- Treat deferred dispatch as at-least-once-capable integration, not an
  exactly-once guarantee. Require deterministic operation ids, idempotent
  dispatch, authenticated callbacks, reconciliation, and an orphan policy.
- Treat `ActionRun` as runtime lifecycle truth. Flower checkpoints, events,
  signals, futures, and callback payloads are orchestration or delivery data.

## Source Of Truth

Prefer the checked-out runtime source and its tests when this guide and an API
disagree. Useful upstream documents include:

```text
flower-action-runtime/README.md
flower-action-runtime/flower-action-runtime-core/README.md
flower-action-runtime/docs/architecture/DEFERRED_ACTION_EXECUTION.md
flower-action-runtime/docs/architecture/ACTION_RUN_PERSISTENCE.md
flower-action-runtime/docs/architecture/EXECUTION_BACKEND_STRATEGY.md
flower-action-runtime/docs/architecture/V0_2_MIGRATION_AND_MODULE_IMPACT.md
flower-action-runtime/docs/architecture/V0_3_MIGRATION_AND_MODULE_IMPACT.md
```
