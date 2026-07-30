# Host Build And Module Selection

Use this reference when creating or changing a Maven or Gradle host, choosing
Action Runtime modules, or resolving Flower compatibility. Treat it as a
decision map, not a required bundle.

## Contents

- [Selection Rules](#selection-rules)
- [Published Modules](#published-modules)
- [Common Compositions](#common-compositions)
- [Maven Setup](#maven-setup)
- [Gradle Kotlin DSL Setup](#gradle-kotlin-dsl-setup)
- [JDBC Persistence](#jdbc-persistence)
- [Verification](#verification)

## Selection Rules

1. Add Action Runtime only when a business side effect needs an explicit
   proposal, validation, policy, approval, idempotency, execution, or audit
   boundary. A Flower workflow alone does not require Action Runtime.
2. Inspect the host build, Java level, Flower execution model, database, and
   test setup before selecting modules.
3. Preserve compatible host dependency management and add only modules
   justified by the requirements.
4. Keep every Action Runtime module on `0.3.1`. When the host directly declares
   the Flower framework artifacts `flower-core` or `flower-eventloop`, align
   those framework artifacts to Flower `0.1.1`.
5. Use Maven Central; do not substitute a SNAPSHOT, `mavenLocal()`, or a
   neighboring source build.
6. If host code directly imports a module's public API, declare that module
   directly instead of relying only on an incidental transitive dependency.
7. State the modules selected, the requirements that selected them, and the
   optional modules deliberately omitted.

Action Runtime `0.3.1` requires Java 21 or newer. There is no Action Runtime
Spring Boot starter in this release; wire registry, policy, stores, executors,
runtime, and any Spring beans explicitly.

## Published Modules

All application-facing coordinates use group `io.github.flowerjvm` and version
`0.3.1`.

| Requirement | Artifact | Add when | Leave out when |
| --- | --- | --- | --- |
| Direct governed-action pipeline | `flower-action-runtime-core` | Any Action Runtime control boundary is required | No governed action is required |
| Flower Flow/Step backend | `flower-action-runtime-workflow` | The shared pipeline stages need Flower Flow/Step visibility | Direct runtime is sufficient; do not use it as a durable-wait backend |
| JDBC `ActionRun` persistence | `flower-action-runtime-persistence-jdbc` | JDBC is the chosen durable `RunStore` | A custom shared store or a same-JVM local/test `InMemoryRunStore` is sufficient |
| Event-loop approval/resume backend | `flower-action-runtime-eventloop` | Approval waits, deadlines, and resume justify the experimental Flower EventFlow backend | Direct/workflow execution is sufficient; do not use it as a generic async executor |

`workflow` brings Action Runtime core and `flower-core:0.1.1`.
`eventloop` brings Action Runtime core and `flower-eventloop:0.1.1`.
`persistence-jdbc` brings Action Runtime core but no production JDBC driver.
Do not add both Flower backends by default; select each only for a demonstrated
execution need.

Action Runtime `0.3.1` is pre-1.0 and its APIs may change. The
`flower-action-runtime-eventloop` backend is experimental. Pin exact versions
and validate that backend's operational behavior before selecting it for
production.

## Common Compositions

| Need | Minimum composition |
| --- | --- |
| Synchronous governed action | `flower-action-runtime-core` |
| Short-lived same-JVM async, approval, or cancellation | Core plus a queryable `InMemoryRunStore` may be sufficient for local/test use |
| Restart-sensitive, multi-instance, or production deferred action | Core plus a queryable shared durable `RunStore`; add `persistence-jdbc` only when JDBC is the selected store |
| Flow/Step visibility for the control pipeline | Core plus `flower-action-runtime-workflow` |
| Event-driven approval/deadline/resume | Core plus `flower-action-runtime-eventloop` and a queryable `RunStore`; use shared durability when restart or multi-instance recovery is required |
| JDBC-backed event-loop action | Core plus `flower-action-runtime-eventloop` plus `flower-action-runtime-persistence-jdbc` |

Approval, async, deferred completion, callbacks, cancellation, recovery, and
multi-instance work must not use `RunStore.noop()`. The workflow backend
provides Flow/Step visibility but does not turn transient waiting into durable
waiting. `InMemoryRunStore` remains process-local; it cannot prove restart or
multi-instance recovery.

## Maven Setup

Maven Central is the default repository, so a released consumer needs no custom
`<repositories>` entry.

```xml
<properties>
    <flower-action-runtime.version>0.3.1</flower-action-runtime.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.github.flowerjvm</groupId>
        <artifactId>flower-action-runtime-core</artifactId>
        <version>${flower-action-runtime.version}</version>
    </dependency>
</dependencies>
```

Add only the selected optional artifacts from the table, using the same
`${flower-action-runtime.version}` property. Keep database drivers and test
frameworks under the host's existing dependency management.

## Gradle Kotlin DSL Setup

```kotlin
repositories {
    mavenCentral()
}

val flowerActionRuntimeVersion = "0.3.1"

dependencies {
    implementation(
        "io.github.flowerjvm:flower-action-runtime-core:$flowerActionRuntimeVersion"
    )

    // Add only when a requirement selects the module:
    // implementation("io.github.flowerjvm:flower-action-runtime-workflow:$flowerActionRuntimeVersion")
    // implementation("io.github.flowerjvm:flower-action-runtime-persistence-jdbc:$flowerActionRuntimeVersion")
    // implementation("io.github.flowerjvm:flower-action-runtime-eventloop:$flowerActionRuntimeVersion")
}
```

A Groovy DSL host should express the same repository, coordinates, versions,
and scopes in Groovy syntax.

## JDBC Persistence

`flower-action-runtime-persistence-jdbc` accepts a host-provided `DataSource`
and provides `JdbcRunStore`. It does not provide a production database driver,
connection pool, `DataSource`, migration runner, Spring auto-configuration, or
automatic schema creation.

Fresh schema resources are:

```text
db/action_run/h2.sql
db/action_run/mysql.sql
db/action_run/postgresql.sql
```

Versioned migration resources are under `db/action_run/migration/`. Apply the
matching scripts through the host's migration boundary; never edit an already
applied migration. Treat H2 as a valid test/local dialect, not proof of native
MySQL or PostgreSQL production validation.

The shipped schema/dialect coverage is H2, MySQL, and PostgreSQL only. Another
database requires a separately validated custom schema and store; do not infer
support merely because it has a JDBC driver.

JDBC CAS prevents stale state overwrite. It does not provide queue ownership,
leasing, an outbox, or exactly-once external effects.

## Verification

Run the host's real wrapper or installed build tool:

```powershell
mvn -B -ntp verify
```

```powershell
.\gradlew check
```

On POSIX:

```bash
./gradlew check
```

Inspect the dependency tree for Action Runtime `0.3.1`, Flower `0.1.1`, and
host-managed Jackson convergence. When the host directly authors or configures
Flower Flows, Steps, Workers, waits, or checkpoints, also follow
`flower-app-guide`, configure Flower Check, and keep deterministic Flow tests.
Merely consuming a prebuilt Action Runtime backend does not by itself require
host-authored Flow tests. Never report an unexecuted command as passing.
