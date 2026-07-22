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

The current guide targets the published `flower-action-runtime 0.3.0` release
and keeps migration context for 0.1/0.2 applications. Load the related
[`flower-app-guide`](https://github.com/flowerjvm/flower-agent-skills/tree/main/agent-skills/flower-app-guide)
as well when changing Flower Flows, Steps, Guards, Workers, waits, or
checkpoint/resume behavior.

Consumer dependencies must use the exact Maven Central release, for example:

```kotlin
implementation("io.github.flowerjvm:flower-action-runtime-core:0.3.0")
```

Add `flower-action-runtime-persistence-jdbc`,
`flower-action-runtime-workflow`, or `flower-action-runtime-eventloop` at the
same `0.3.0` version only when the host needs that module. This skill contains
documentation only: it does not depend on a runtime checkout, `mavenLocal()`,
or SNAPSHOT artifacts.

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

## License

Licensed under the Apache License 2.0. See `LICENSE` and `NOTICE`.
