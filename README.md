# Flower Action Runtime Guide

Agent skill for building and reviewing governed business actions with
[`flower-action-runtime`](https://github.com/flowerjvm/flower-action-runtime).

The guide covers:

- action proposals, definitions, policy, approval, and pre-execution guards;
- synchronous, in-process asynchronous, and durable deferred execution;
- `ActionRun`, `RunStore`, JDBC persistence, versioned CAS, and recovery;
- completion/cancellation races and authenticated callbacks;
- direct, Flower workflow, and Flower event-loop backend selection;
- host integration, parity tests, and real concurrency tests.

The current guide targets `flower-action-runtime 0.2.0`. Load the related
[`flower-app-guide`](https://github.com/flowerjvm/flower-agent-skills/tree/main/agent-skills/flower-app-guide)
as well when changing Flower Flows, Steps, Guards, Workers, waits, or
checkpoint/resume behavior.

## Install

Install this repository root with an Agent Skills-compatible installer, or
clone/link it into the agent's user-level skill directory under the name
`flower-action-runtime-guide`. The repository root intentionally contains
`SKILL.md` so it can be installed directly.

Invoke it explicitly as:

```text
$flower-action-runtime-guide
```

Agents may also select it automatically when a task matches the description in
`SKILL.md`.

## Contents

- `SKILL.md`: task routing, workflow, and non-negotiable safety rules.
- `agents/openai.yaml`: Codex-facing display metadata and default prompt.
- `references/`: focused guidance for controls, execution modes, persistence,
  Flower backends, testing, host integration, and verification.

Start with `references/00-guide-version.md`,
`references/01-runtime-quick-rules.md`, and `references/90-verification.md`,
then follow the reference routing in `SKILL.md`.
