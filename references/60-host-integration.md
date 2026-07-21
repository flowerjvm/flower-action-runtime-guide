# Host Integration

## Keep Adapters Thin

REST controllers, MCP tools, schedulers, message listeners, and AI planners
should authenticate input, construct an `ActionProposal`, invoke the runtime,
and map the returned result. They must not call the domain mutation directly.

Keep the domain operation in an application/domain service called by the
registered executor. This preserves one side-effect boundary regardless of how
many request channels exist.

## Build Trusted Context

Resolve principal, roles, tenant or office, and request channel from trusted
host infrastructure. The proposer type may describe an AI or system proposer,
but it does not grant authority. Policy evaluates the execution principal and
current resource facts.

Create a unique run id for each accepted request. Reuse an idempotency key for
retries of the same logical operation. Do not reuse the run id as the
idempotency key or create a second run for a duplicate callback.

## Persistence And Transactions

Use a durable store when a result can outlive the request or process. Apply the
runtime's database migrations through the host's migration system and confirm
the schema owner and tenant isolation policy.

If an executor changes domain state and emits an event, use the host's normal
transactional outbox or after-commit mechanism. The runtime cannot make an
arbitrary database mutation and external message globally atomic.

Expose status by reading the persisted `ActionRun`; do not infer completion
from HTTP connection state, a local future, or a queue message alone.

## Deferred Callback Boundary

Authenticate callbacks independently of the original request. Resolve the
tenant from trusted callback credentials or stored run ownership, then verify
run id, attempt token, external operation identity, and terminal result shape.
Invoke the runtime completion API and return its consistent stored outcome.

Rate-limit callbacks and record rejected attempts without leaking whether a
run exists in another tenant. Do not accept a caller-supplied tenant id as
sufficient authorization.

## Operational Readiness

Provide metrics for proposals, control rejections, approval latency, dispatch
latency, waiting runs, terminal outcomes, CAS conflicts, stale callbacks,
executor saturation, and recovery actions. Alert on growing non-terminal age
and repeated reconciliation failures.

