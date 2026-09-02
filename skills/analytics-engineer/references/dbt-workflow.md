# dbt Workflow

## Contents

- [Discover the project contract](#discover-the-project-contract)
- [Design the change](#design-the-change)
- [Implement all resources together](#implement-all-resources-together)
- [Isolate concurrent development](#isolate-concurrent-development)
- [Validate by environment and row](#validate-by-environment-and-row)
- [Retire resources completely](#retire-resources-completely)
- [Prepare a reviewable handoff](#prepare-a-reviewable-handoff)

## Discover the project contract

- Read repository instructions, `dbt_project.yml`, package declarations,
  profiles guidance, linter configuration, CI jobs, and the pull-request
  template.
- Determine the supported dbt and adapter versions, model paths,
  materializations, targets, variables, tags, selectors, and state/defer or
  clone workflow.
- Inspect the target SQL plus schema YAML, docs blocks, macros, tests, sources,
  seeds, snapshots, semantic models, metrics, exposures, and downstream refs.
- Check for a linked task or specification and preserve its identifier in branch
  or PR metadata only when the user requests those git operations.

## Design the change

- Declare model purpose, grain, unique key, materialization, refresh behavior,
  and contract impact.
- Reuse existing refs and macros before adding dependencies.
- Place shared transformations upstream at the layer that owns the concept,
  while avoiding gratuitous intermediate models and DAG fan-out.
- For incremental models, specify late-arriving update, deletion, lookback,
  schema-change, and full-refresh behavior.
- For source migrations, use a configured cutoff when available, mutually
  exclusive eras, and normalized cross-system keys.

## Implement all resources together

- Add schema YAML, descriptions, and tests for every new model.
- Test the true unique key or unique column combination. Use a surrogate key
  when it is a deliberate consumer contract, not merely to hide duplicates.
- Update semantic models, metrics, exposures, docs, and downstream BI mappings
  for contract changes.
- Keep `ref()` and `source()` calls in named top-level CTEs when local convention
  requires it.
- Keep calculations in intermediate CTEs and use the final CTE to assemble the
  output when practical.

## Isolate concurrent development

Before a warehouse-writing command, determine whether another branch, task, or
agent may be using the default development schema. If so, read
[concurrent-dbt-development.md](concurrent-dbt-development.md) and assign this
task its own target schema, target path, and log path. Use a separate git
worktree when concurrent tasks have different code changes.

Carry the environment identity explicitly on every command. Do not rely on a
shell export persisting across invocations or silently fall back to the default
personal schema.

## Validate by environment and row

Use exact commands documented by the repository.

### Development validation

1. Parse or compile the changed selector and lint modified SQL and YAML.
2. Preserve a readable production baseline for each changed model. Clone or
   defer unchanged production parents into the task-specific development
   environment when supported, then rebuild only the changed selector and
   required parents.
3. Run targeted model, relationship, and custom data tests in development.
4. Compare the development relation with the current production relation at
   the model's declared unique key:
   - limit both sides to the same time and business scope;
   - compare shared columns with null-safe equality;
   - use explicit tolerances only for columns whose numeric behavior warrants
     them;
   - exclude or normalize nondeterministic audit columns deliberately;
   - use a full outer join to classify development-only, production-only,
     changed, and exact-match keys;
   - report category counts and inspect representative rows from each material
     difference class.
5. Reconcile row counts, uniqueness, null rates, and additive measures. Validate
   newly introduced columns separately when they have no production analogue.
6. Build affected children or validate BI contracts when the interface changed.

A comparison need not produce zero differences. Connect expected differences
to the requested business change and investigate every unexplained material
difference. Avoid comparing a development-only rolling window with full
production history; apply the same predicate to both relations.

### Production validation

After an authorized deployment through the repository's normal orchestration:

1. run targeted production tests and grain checks;
2. verify key uniqueness, row counts, null rates, and business invariants on the
   deployed relation;
3. compare affected rows and measures with an approved expected result or a
   captured pre-deployment baseline;
4. smoke-test affected downstream models, dashboards, or applications;
5. complete or report any documented full refresh, object removal, backfill, or
   communication task.

Do not run against production, perform an expensive full refresh, or rebuild
the entire graph without evidence that it is required and authorization for the
environment impact. Prefer targeted selectors over full-project runs.

Record the task schema, target/log paths, exact relation names, commit or
artifact state, comparison scope, and checks that could not run because
credentials, warehouse access, dependencies, or fixtures were unavailable.
Never substitute successful compilation for successful data validation.

## Retire resources completely

When deleting or deprecating a model:

- find every `ref()`, test, schema entry, docs block, semantic model, metric,
  exposure, selector, macro assumption, and BI reference;
- remove active metadata that would keep the model in docs or lineage;
- migrate or remove downstream consumers;
- preserve the retired SQL's formatting and history unless a rewrite is part of
  the request;
- verify that parsing no longer exposes stale active resources.

## Prepare a reviewable handoff

- Explain why the chosen layer owns the logic and how grain is preserved.
- Describe compatibility, backfill, migration-boundary, and downstream BI
  effects.
- List exact validation commands and results.
- Read [pull-request-workflow.md](pull-request-workflow.md), then use the
  repository PR template and target branch when asked to comment or open a PR.
- Do not commit, push, open a PR, deploy, or update task systems unless the user
  authorized that action.
