# Full Development-versus-Production Validation

## Contents

- [Set the validation scope](#set-the-validation-scope)
- [Protect the comparison](#protect-the-comparison)
- [Build and test development](#build-and-test-development)
- [Compare changed columns](#compare-changed-columns)
- [Accept or escalate differences](#accept-or-escalate-differences)
- [Downstream validation is opt-in](#downstream-validation-is-opt-in)
- [Apply the quality gate](#apply-the-quality-gate)
- [Record evidence](#record-evidence)

## Set the validation scope

Full validation means completing the required checks for changed columns in
the directly changed models. It does not mean testing every column or following
the downstream graph.

Before warehouse execution, record a model-to-column allowlist from the ticket,
diff, and relevant SQL or macro logic. Include added, removed, renamed, retyped,
or logic-changed output columns. Explain each column's inclusion; file edits or
formatting alone do not make every output column changed. For a new model, all
output columns are new and therefore in scope. For shared macro or ephemeral
changes, name the intended outputs explicitly; do not automatically select all
consumers. If testing requires a downstream relation, wait for an explicit
request before executing that validation.

Unchanged columns are outside the default test scope. Read keys and upstream
inputs needed to align rows or calculate expected values, but do not profile or
assert unchanged output values. Check key uniqueness only as needed to ensure
that the row comparison is valid. Do not expand to all columns because the
model's SQL must be rebuilt as a whole.

Record downstream scope as `not requested` by default. A request for "full
validation" alone does not opt into downstream testing. Explicitly requested
downstream checks must identify the models and columns to test; do not infer an
all-column comparison from permission to test a downstream model.

Reporting dependency deprecations additionally require a live content-impact
check under [reporting-impact.md](reporting-impact.md), even when the changed
column allowlist is empty. This is separate from downstream data validation.

## Protect the comparison

- Keep production read-only during development validation.
- Use a task-specific schema and separate production-state, target, and log
  paths. Confirm resolved relations before warehouse writes.
- Tie the allowlist and results to the current commit and working-tree
  fingerprint. Re-evaluate the allowlist after a correction.
- Use the same time and business predicates on development and production.
  Record production baseline timestamps and any source-freshness differences.
- Document normalization, tolerances, and nondeterministic-column exclusions
  only for columns in the allowlist.

## Build and test development

1. Parse the project, compile directly changed models, and lint modified files.
   Record parse and compile outcomes separately; one does not prove the other.
2. Clone or defer unchanged production parents needed to build those models.
   Dependency preparation does not add those parents to the validation scope.
3. Materialize only the named models in the isolated environment. Do not use a
   trailing `+`, descendant selector, or full-project build by default.
4. Select only test nodes whose assertions target the allowlisted changed
   columns, plus any minimal key checks required for comparison. Inspect
   generic, singular, relationship, and unit tests for unrelated assertions.
   Do not run a whole-model suite just because it is attached to that model.
5. Verify selection with `dbt ls` or the manifest before execution. Avoid eager
   indirect selection that pulls in unchanged-column or downstream tests.

Prefer `dbt run` for the named models followed by `dbt test` with an explicit
list of relevant test nodes and `--indirect-selection empty` when supported by
the project's version. A scoped `dbt build` is also acceptable if its resolved
model and test list stays within the allowlist. If no relevant test nodes
exist, use focused comparison queries; never fall back to the entire suite.
Retain applicable tests for unchanged fields in the repository; only their
execution is excluded from this validation run. See dbt's
[indirect selection guidance](https://docs.getdbt.com/reference/global-configs/indirect-selection)
when resolving test selection for the installed version.

## Compare changed columns

For allowlisted columns only, compare presence, name, type, precision, scale,
nullability, and relevant documentation or test changes. Check order only if an
intentional column-order change is part of the consumer contract. Removed
columns require absence checks; renamed columns require an explicit old-to-new
mapping. New columns require business expectations and profiles, not a
fabricated production equivalent.

At the stable row key, use a full outer comparison over the same population and
null-safe equality on the allowlisted comparable columns. Report dev-only and
prod-only keys, and matched keys that differ or match **on those columns**.
Matching selected columns does not establish whole-row or whole-model parity.

For each selected comparable column, report changed-value counts and rates,
null-rate and distinctness changes, safe example keys, and whether differences
match the ticket. Add meaningful numeric deltas, category frequencies, date
boundaries, or other type-specific checks only for selected columns. Project
only keys and selected values into comparison queries; avoid `select *` and
whole-row hashes that silently compare unchanged columns.

Profile added columns against their declared invariants. Use complete
in-scope populations for exact comparisons; disclose sampling or approximate
metrics rather than describing them as exhaustive evidence. Missing required
in-scope evidence prevents a pass. A model with no changed output columns may
need only static checks or materialization; record column comparisons as not
applicable instead of choosing unchanged columns to test.

## Accept or escalate differences

Classify each difference by cause and acceptance, separately from magnitude.
Keep exact counts, deltas, comparison windows, run timestamps, and supporting
queries internally even when the PR only needs a short explanation.

- Rounding differences are generally acceptable when attributable to known
  precision or rounding behavior and within a justified tolerance. Do not
  choose a tolerance after seeing results merely to hide a defect.
- Differences caused solely by model run times (for example, development has
  fresher data) are generally acceptable when timestamps and aligned-window
  checks or row evidence demonstrate the cause. Do not use "freshness" to
  excuse unexplained differences in overlapping history.
- Logic changes that affect metrics require explicit user acceptance of the
  specific behavior and scope, in the ticket or conversation. The developer's
  explanation, small magnitude, or consistency with its implementation is not
  approval. A broad request to refactor is not acceptance of metric drift.
- When the cause, materiality, or acceptance is uncertain, ask the user through
  the coordinator with the metric, production/development values, scope,
  proposed cause, and decision needed. Standalone testers ask directly.
  Continue independent checks, but do not issue a pass for the unresolved
  scope. A known defect is `FAIL`; a pending acceptance decision is `BLOCKED`.

Do not invent a universal percentage threshold for "negligible." For a
negligible, accepted difference, the PR may say "<difference> is negligible
and accepted because <demonstrated reason>" without exact numerical deltas.
For large key-metric differences, show production, development, absolute or
relative change, cause, and acceptance status in a table, even when accepted.
Report matching changed columns concisely in the changed-column table.

## Downstream validation is opt-in

Do not build, test, profile, diff data, or smoke-test downstream models,
columns, metrics, or BI consumers unless the user explicitly requests that
validation. The required API-based dependency/content check for reporting
deprecations is the narrow exception; it does not run downstream data queries.
Reading lineage to understand an implementation does not authorize downstream
execution. Do not treat propagation of an upstream change as an opt-in.

When requested, use an explicit downstream model/column allowlist, run the same
scoped checks, and report the request and results separately. Keep any remaining
descendants excluded. When not requested, report `Downstream validation: not
requested; not performed`. This is an intentional scope exclusion, not a failed
check, incomplete validation, or a blocker for PR preparation.

This opt-in rule includes immediate downstream compile/run/test checks. When
the user requests one-hop coverage, resolve changed models plus their immediate
children explicitly (for example, `model+1`, not an unbounded `model+`) and
inspect the selection before execution. Do not infer detailed downstream value
comparisons or all-column tests from a request for compile/run/test coverage.
In the PR, state that changed models and immediate downstream candidates passed
only when the respective checks actually ran successfully on that scope.
Project compilation performed by `dbt docs generate` for required DAG captures
is documentation preparation; record it separately and do not count it as
requested downstream validation or as evidence of model runs/tests.

## Apply the quality gate

- `PASS`: all required checks for the recorded scope ran, selected-column
  differences are explained and accepted under the rules above, comparison
  keys are usable, and evidence matches
  the current fingerprint. For reporting deprecations, the required live
  impact check must also be resolved under `reporting-impact.md`.
- `FAIL`: a correctable implementation or test defect exists within that scope.
- `BLOCKED`: access, missing data, or unresolved semantics prevent required
  in-scope validation. Never block solely because unchanged columns or
  unrequested downstream resources were not tested.

After a fix, rerun the required checks for the updated changed-column allowlist;
do not broaden validation to unrelated columns or models.

## Record evidence

Include the model/column allowlist, exact relations, key, comparison predicates,
commands, results, safe examples, and artifact paths in the tester packet from
[agent-handoffs.md](agent-handoffs.md). Report unchanged columns as excluded and
downstream validation as not requested unless explicitly included. Keep
post-deployment checks within the same scope unless the user expands it.
Record reporting-deprecation impact separately, including live coverage and
limitations. Keep commands, JSON, fingerprints, and detailed inventories in
the packet; the PR receives a concise findings summary, not the raw packet.
Record parse, compile, run, and test statuses separately, with their scopes,
failure/warning/skip counts, and acceptance sources for differences. Missing
tests or a compile-only check cannot be summarized as a successful test run.

Preserve the isolated schema until PR creation and the user's cleanup decision.
