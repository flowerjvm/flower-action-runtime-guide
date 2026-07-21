# Action Model And Control Boundary

## Model A Request As A Proposal

An entry point may request a side effect, but it must not perform that side
effect directly. Convert REST, UI, MCP, scheduler, batch, and AI planner input
into an `ActionProposal`, then let the registered action pipeline decide
whether execution is allowed.

Use a stable action id such as `document.publish` or `invoice.cancel`. Register
one `ActionDefinition` for that id and keep the actual domain mutation inside
its `ActionExecutor`.

## Keep Identities Separate

Do not compress these values into one generic actor string:

| Concern | Meaning |
| --- | --- |
| Request channel | Where the request entered: UI, API, MCP, scheduler, batch, or another adapter |
| Proposer type | Who proposed it: human, agent, system, or service |
| Execution principal | The authenticated identity whose authority is evaluated |
| Tenant or office | The data-ownership boundary used by policy and persistence |
| Run id | The unique lifecycle identity of this request |
| Idempotency key | The logical operation key used to collapse transport retries |

Derive authenticated identity and tenant data from trusted host context. Never
trust an AI-generated payload or an arbitrary callback body for these values.

## Required Control Order

Preserve this order for initial execution and every resumed execution:

```text
registry resolution
  -> input validation
  -> policy evaluation
  -> approval decision
  -> duplicate handling
  -> pre-execution guard
  -> executor dispatch
  -> result persistence and audit
```

Approval is not a permanent authorization grant. Between proposal and approval,
the tenant, resource, role, account state, or action definition may have
changed. On approval resume, resolve the definition again, validate again,
evaluate policy again, and run `PreExecutionGuard` immediately before dispatch.

Use `PreExecutionGuard` for last-moment facts that must be true at the side
effect boundary, such as resource version, current ownership, account lock, or
an external safety condition. Keep durable business invariants in the domain
service as well.

## Results Are A Machine Contract

Use `ActionExecutionStatus` for the broad outcome and
`ActionExecutionResult.code()` for a stable, machine-readable reason. Use
`RetryDisposition` to tell callers whether retrying is meaningful. Human
messages are for operators and users; clients must not parse them to make
workflow decisions.

Examples of useful result-code families include validation rejection, policy
denial, approval pending or rejected, duplicate request, dispatch failure,
external timeout, stale attempt, cancellation, and domain-specific rejection.
Keep codes stable across wording and localization changes.

## Audit And Trace

Audit the proposal, every control decision, approval transition, dispatch,
completion, cancellation, and failure with the run id and tenant boundary.
Trace data helps diagnose execution; audit data explains who requested what,
which controls decided it, and what terminal result was recorded. Do not treat
ordinary application logs as the durable audit record.
