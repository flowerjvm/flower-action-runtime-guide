# Flower Action Runtime Guide Version

Guide version: `0.4.1`

Last updated: `2026-07-30`

Target runtime line: `flower-action-runtime 0.3.1`

Latest published runtime release: `flower-action-runtime 0.3.1`

Published baselines covered by migration guidance: `0.1.0` and `0.2.0`

Related Flower application guide: `flower-app-guide 0.3.0`

Verified Flower backend compatibility: `flower-core 0.1.1` and
`flower-eventloop 0.1.1`

Scope:

- controlled business-action boundaries
- Maven/Gradle host setup, published coordinates, and requirement-driven module
  selection
- policy, approval, pre-execution checks, visibility-safe owner-aware duplicate
  handling, audit, and trace
- synchronous, async, and deferred dispatch
- durable ActionRun state, JDBC CAS, completion, cancellation, and recovery
- direct, Flower workflow, and Flower event-loop backend selection
- host integration and deterministic verification

Version policy:

- Patch: clarify guidance or make its required verification evidence more
  explicit without changing a control rule.
- Minor: add runtime/module guidance or a compatible workflow.
- Major: change a host-facing control or safety rule incompatibly.

The runtime is pre-1.0. Consumer examples must use the published `0.3.1`
artifacts and be checked against the matching `v0.3.1` source tag. A mutable
runtime checkout may be on a later SNAPSHOT and is not a consumer dependency.
