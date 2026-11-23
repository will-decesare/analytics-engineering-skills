# dbt Workflow

## Contents

- [Discover the project contract](#discover-the-project-contract)
- [Start from current remote main](#start-from-current-remote-main)
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
- Capture the motivation separately from the requested implementation. If the
  ticket and user messages do not explain why, ask through the coordinator
  while continuing independent work; do not invent the PR rationale.

## Start from current remote main

Before creating any task branch from a production base, including a branch in
a new worktree, the developer must verify that the local starting point is
current with the remote. Apply this independently in every affected repository
(for example, dbt and its companion BI repository).

1. Resolve the authoritative remote and base branch from repository guidance
   and Git configuration. Default to `main`; honor an explicitly selected
   production/release base. Do not assume a fork's `origin` is authoritative
   when the project uses a different upstream. Ask if the choice is ambiguous.
2. Inspect working-tree status and `git worktree list` before switching or
   updating a local branch. Fetch the selected remote base immediately before
   branching; do not trust a cached remote-tracking ref or an old "up to date"
   status. A targeted fetch can use
   `git fetch <remote> refs/heads/<base>:refs/remotes/<remote>/<base>`.
   If fetching fails or the remote base is missing, stop branch creation and
   report the blocker rather than proceeding from stale local state.
3. Compare the local base and fetched remote base, including their exact tips
   and ahead/behind counts. For example,
   `git rev-list --left-right --count <base>...<remote>/<base>` reports
   local-only commits first and remote-only commits second.
4. If the tips match, the base is current. If local is only behind, update it
   with `git merge --ff-only <remote>/<base>` in its clean, safely available
   base-branch checkout, then verify both tips match. Never run that update on
   the current feature branch by accident. If local is ahead or diverged, stop
   and ask how to handle its local-only commits; do not silently include them,
   create a merge commit, rebase, reset, or force-update the base.
5. Preserve uncommitted changes and other tasks' checkouts. If a behind/equal
   base checkout is dirty or in active use, create the new task's isolated
   worktree directly from the freshly fetched remote-base tip instead of
   updating that checkout. The same option applies when no local base branch
   exists. Report that the task starts from current remote code while the
   other local checkout was left unchanged; do not claim it was synchronized.
   If isolation is unavailable, ask before proceeding. Do not stash, discard,
   or carry unrelated edits into the new branch automatically.
6. Create the branch using the verified tip as an explicit starting point,
   not an arbitrary current `HEAD`. Check the new branch's initial `HEAD`
   equals that tip before editing. Record remote, base, fetch time, comparison
   outcome, and starting commit in the internal Developer Handoff Packet.

For an existing PR, resolve its named branch and worktree before editing and
reuse a suitable checkout. Do not recreate or reset it. If isolation is needed,
make the edits reviewable on the intended branch before handoff without
committing or overwriting unrelated work. Report the absolute checkout path;
uncommitted edits on a detached HEAD are not edits on the named branch.
This preflight does not authorize merging or rebasing tested feature code.

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

Use exact commands documented by the repository with the mode and column scope
from [full-validation.md](full-validation.md). Reporting refactors compare all
comparable outputs in directly changed models; focused fixes select changed
outputs. Downstream data validation requires an explicit request. Reporting deprecations additionally require the
live content-impact check below. Apply the same data-testing boundary to
development and any authorized post-deployment checks.

Under a no-refresh instruction, do not clone fresh data or rebuild relations.
Use existing snapshots or read-only inline candidate SQL as specified in
full-validation.md; label unperformed materialization/tests rather than claiming
they ran. Documentation-only work does not require this build ladder.

### Development validation

Follow [full-validation.md](full-validation.md#build-and-test-development) for
relation preparation, runtime preflight, scoped execution, deployment scenarios,
and row-keyed comparisons. Keep the relation plan distinct from the comparison
allowlist. Use the recorded baseline and metric definitions throughout repairs;
apply [difference acceptance](full-validation.md#accept-or-escalate-differences)
before declaring a pass. An explained difference is not automatically accepted.

### Production validation

After an authorized deployment through the repository's normal orchestration:

1. run production tests targeting the approved mode and column scope;
2. use keys to align rows and check nulls or business invariants only for
   selected columns;
3. compare those columns with the approved expected results or captured
   pre-deployment baseline;
4. smoke-test downstream models, columns, dashboards, or applications only
   when explicitly requested;
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

- when it is a reporting dependency, follow
  [reporting-impact.md](reporting-impact.md) and use relevant reporting APIs to
  check live dashboards, visualizations, and other saved content;
- find every `ref()`, test, schema entry, docs block, semantic model, metric,
  exposure, selector, macro assumption, and BI reference;
- remove active metadata that would keep the model in docs or lineage;
- migrate or remove in-scope repository consumers; seek authorization before
  modifying hosted reporting content and report outstanding migration work;
- preserve the retired SQL's formatting and history unless a rewrite is part of
  the request;
- verify that parsing no longer exposes stale active resources.

## Prepare a reviewable handoff

- Apply [review-checklist.md](review-checklist.md) before handoff: classify
  review findings and remove work that no longer supports the final objective.
- Explain why the chosen layer owns the logic and how grain is preserved.
- Describe compatibility, backfill, migration-boundary, and downstream BI
  effects.
- Keep exact validation commands and results in the internal handoff; summarize
  reviewer-relevant findings in the PR without hashes or artifact dumps.
- Preserve the before-state DAG and capture actual dbt Docs before/after
  screenshots for dependency changes under [dag-screenshots.md](dag-screenshots.md).
- Read [pull-request-workflow.md](pull-request-workflow.md), then use the
  repository PR template and target branch when asked to comment or open a PR.
- Do not commit, push, open a PR, deploy, or update task systems unless the user
  authorized that action.
