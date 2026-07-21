# Execution Modes

Choose the executor contract from the real lifetime and ownership of the work.
Do not wrap a blocking action in `CompletableFuture` merely to make the API look
asynchronous.

| Contract | Use when | Lifecycle result |
| --- | --- | --- |
| `ActionExecutor` | Work is short, bounded, and may safely finish in the caller's execution budget | Terminal result returned inline |
| `AsyncActionExecutor` | Work is in-process and asynchronous, but the current process still owns completion | Completion stage eventually produces the terminal result |
| `DeferredActionExecutor` | An external worker, queue, remote service, long-running job, or later callback owns completion | Dispatch returns an operation descriptor; the run becomes `WAITING_EXTERNAL` |

## Synchronous Execution

Use synchronous execution only for bounded work. A synchronous run that is
already `RUNNING` cannot truthfully be marked cancelled while its side effect
continues. If meaningful cancellation or long duration is required, use an
async or deferred design with a cooperative cancellation hook.

## In-Process Async Execution

Use a host-managed, bounded executor. Share it according to the application's
resource policy and expose saturation through metrics. Never create one thread
pool per action, use an unbounded queue without an explicit capacity decision,
or block a Flower tick with `Future.get()` or `join()`.

An async stage does not make the work durable. If the process can restart while
work is running, either provide recovery that can reconstruct the attempt or
use deferred execution backed by an external durable system.

## Durable Deferred Execution

`DeferredActionExecutor` should return a dispatch descriptor containing the
external operation identity and useful timing or metadata. Persist the
transition to `WAITING_EXTERNAL` before treating dispatch as accepted.

Complete the run through `CompletableActionRuntime.complete(...)`, supplying
the run id, current attempt token, and terminal result. Reject stale or forged
attempt tokens. A duplicate completion for the already-recorded terminal
attempt should return a consistent idempotent result rather than execute a
second side effect.

The callback adapter must authenticate the sender, enforce tenant ownership,
validate the run id and attempt token, and map only a terminal external result.
It must not bypass the runtime by writing `ActionRun` directly.

## Cancellation

Cancellation is a state transition plus, when supported, a cooperative request
to the active executor or external operation. Make the external cancellation
hook idempotent for the tuple of run id, attempt token, and operation id.

Concurrent completion and cancellation are normal. Versioned CAS decides the
single terminal winner. The losing request must observe and report the stored
terminal truth; it must not overwrite it or issue a second domain mutation.

