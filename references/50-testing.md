# Testing

## Pipeline Tests

Cover at least:

- unknown action and invalid input;
- policy allowed and denied;
- approval not required, pending, approved, rejected, and expired;
- changed policy or invalid input after approval;
- pre-execution guard rejection immediately before dispatch;
- duplicate idempotency behavior;
- stable result code and retry disposition;
- synchronous, async success/failure, and deferred dispatch;
- valid, stale, forged, duplicate, and cross-tenant completion callbacks;
- cancellation before dispatch, while waiting externally, and after terminal
  completion;
- audit events for every important decision and transition.

## Backend Parity

Run equivalent scenarios through direct and Flower backends. Compare control
decisions, persisted statuses, result codes, and audit semantics. Backend
selection must not weaken validation, policy, approval revalidation, or guards.

## Real Concurrency Tests

Two sequential CAS calls from one test thread prove stale-version rejection;
they do not exercise contention. Add real concurrency tests using a bounded
thread pool, barriers or latches, and deterministic timeouts.

For each race, start all contenders from the same stored version and assert:

- exactly one transition wins;
- the stored version advances once for the contested transition;
- all losers observe a consistent terminal result;
- no unexpected exception, deadlock, or timeout occurs;
- the domain completion or cancellation hook is invoked no more than intended.

Exercise complete-versus-complete, cancel-versus-cancel, and
complete-versus-cancel. For JDBC, use separate connections and, when the host
can have multiple instances, two runtime objects sharing the same database.
Repeat the focused concurrency class enough times to catch accidental timing
dependencies, but coordinate starts with barriers rather than sleeps.

## Persistence And Recovery Tests

Test create uniqueness, CAS version rules, serialization of every field,
schema migrations, restart recovery, stale attempt tokens, and terminal-run
idempotency. H2 is useful for fast feedback but does not replace native database
tests for dialect, locking, and transaction behavior.
