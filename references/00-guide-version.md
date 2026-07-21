# Flower Action Runtime Guide Version

Guide version: `0.1.0`

Last updated: `2026-07-22`

Target runtime release: `flower-action-runtime 0.2.0`

Latest published baseline covered by migration guidance: `0.1.0`

Related Flower application guide: `flower-app-guide 0.2.2`

Scope:

- controlled business-action boundaries
- policy, approval, pre-execution checks, duplicate handling, audit, and trace
- synchronous, async, and deferred dispatch
- durable ActionRun state, JDBC CAS, completion, cancellation, and recovery
- direct, Flower workflow, and Flower event-loop backend selection
- host integration and deterministic verification

Version policy:

- Patch: clarify guidance without changing a rule.
- Minor: add runtime/module guidance or a compatible workflow.
- Major: change a host-facing control or safety rule incompatibly.

The runtime is pre-1.0. Inspect the checked-out API and migration notes before
copying examples into a host application.
