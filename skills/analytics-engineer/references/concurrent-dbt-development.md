# Concurrent dbt Development

## Contents

- [Decide whether isolation is required](#decide-whether-isolation-is-required)
- [Name the environment](#name-the-environment)
- [Make the target schema dynamic](#make-the-target-schema-dynamic)
- [Isolate local dbt state](#isolate-local-dbt-state)
- [Populate from production](#populate-from-production)
- [Build and test](#build-and-test)
- [Validate and hand off](#validate-and-hand-off)
- [Clean up safely](#clean-up-safely)

## Decide whether isolation is required

Create a task-specific development environment whenever another branch, task,
developer, automation, or agent may write to the default development schema
before the current work finishes.

Schema isolation protects warehouse relations only. Also isolate:

- code with a separate git worktree when branches differ;
- compiled SQL and JSON artifacts with a unique `--target-path`;
- logs with a unique `--log-path`;
- production state artifacts from development output artifacts.

Inspect repository instructions, `profiles.yml` guidance, custom schema macros,
adapter support, grants, and cleanup policy before creating anything.

## Name the environment

Use a deterministic, collision-resistant, warehouse-safe name such as:

```text
DBT_<USER>_<YYYYMMDD>_<TASK_SLUG>
```

For example:

```text
DBT_ANALYST_20260828_ORDER_COUNTS
```

Do not use the date alone when multiple tasks can start on the same day. Keep
the task slug short, sanitize punctuation, respect identifier-length limits,
and record the selected schema before running dbt.

## Make the target schema dynamic

Prefer a repository-supported mechanism. For dbt Core, a common one-time local
profile configuration is:

```yaml
outputs:
  dev:
    schema: "{{ env_var('DBT_DEV_SCHEMA', 'DBT_<USER>') }}"
```

Do not edit a user's profile without authorization. If the dev output hardcodes
one schema and no supported override exists, stop before warehouse writes and
propose the env-var-backed configuration or a dedicated target output.
dbt Core has no general `--schema` CLI option; do not invent one or assume that
`--target` changes a hardcoded schema.

Check `generate_schema_name`. Its development behavior must retain
`target.schema`; otherwise custom-schema models can escape the task namespace
and collide with other work. A model configured with a custom schema may land
in `<task_schema>_<custom_schema>`, so track every resolved schema rather than
assuming only one is created.

Set the full schema value on every dbt invocation, or use a task-specific
wrapper or profile. Do not rely on an `export` from a different shell process.

## Isolate local dbt state

Choose a task-owned environment root outside shared `target/` and `logs/`:

```text
<environment-root>/
├── prod-state/
├── dev-target/
└── logs/
```

Keep the production state path distinct from the development target path. This
prevents one command from overwriting the manifest used by `--state` and keeps
parallel invocations from racing on `manifest.json` or `run_results.json`.

## Populate from production

Use the repository's documented clone or deferral workflow. A typical dbt Core
sequence is:

```text
dbt parse --target prod \
    --target-path <environment-root>/prod-state \
    --log-path <environment-root>/logs/prod

DBT_DEV_SCHEMA=<task-schema> dbt clone --target dev \
    --state <environment-root>/prod-state \
    --select +<changed-model> \
    --target-path <environment-root>/clone-target \
    --log-path <environment-root>/logs/clone
```

Before running it, confirm that the active dbt executable and adapter match the
versions supported by the project. Inspect the planned selector with `dbt ls`
and compile into an isolated target path to confirm the resolved database and
schema. Abort if resolution points at the default personal schema, production,
or another task's namespace.

Adjust selector syntax to local guidance. Clone the changed model's required
ancestors or use deferral for dependencies so rebuilt refs resolve safely. A
new model will not exist in production state, but its existing parents can
still be cloned or deferred. Inspect graph selectors before execution because
`+<changed-model>` may include substantially more lineage than intended; use a
bounded depth or explicit list when appropriate.

On adapters with zero-copy cloning, `dbt clone` can create inexpensive table
clones. For unsupported relation types or platforms, it may create pointer
views. Treat the target as writable development state in either case and keep
the production source read-only.

`dbt clone` does not replace relations that already exist in the target by
default. Prefer a fresh task schema. Use `--full-refresh`, schema replacement,
or another destructive refresh only when the repository requires it and the
user authorizes the exact target.

## Build and test

Repeat the same task schema and isolated paths for every command:

```text
DBT_DEV_SCHEMA=<task-schema> dbt build --target dev \
    --select <changed-model>+ \
    --state <environment-root>/prod-state \
    --target-path <environment-root>/dev-target \
    --log-path <environment-root>/logs/build
```

Use the narrowest selector that proves the change. Confirm the resolved target
before execution and verify afterward that every created relation belongs to
the task namespace. Abort if a command resolves to the default personal schema
or production unexpectedly.

Run row-grain tests and development-versus-production comparisons against the
task relation. Apply identical time and business predicates to both sides.

## Validate and hand off

Report:

- task schema and any derived custom schemas;
- branch or worktree;
- production state artifact and timestamp;
- cloned, built, and tested selectors;
- target and log paths;
- row-level comparison results and remaining limitations;
- whether cleanup is still required.

Include the isolated schema in PR validation evidence so reviewers can inspect
the correct environment and do not mistake another task's data for this one.

## Clean up safely

Do not drop schemas automatically. Confirm no active task or BI validation uses
the environment, resolve the exact task-owned schema names, and obtain
authorization before cleanup. Drop only the isolated schemas and artifacts
created for that task; never target the default personal schema, production, a
database root, or a wildcard.

After a pull request is successfully created for work that used an isolated
development schema, return the exact schema names and retention considerations
to the analytics-engineer coordinator. The coordinator explicitly asks the user
whether to drop them. Treat an affirmative answer as authorization only for
those named, task-owned schemas; re-check that they are not in active use
immediately before dropping them.
