# Full Development-versus-Production Validation

## Contents

- [Set the validation scope](#set-the-validation-scope)
- [Protect the comparison](#protect-the-comparison)
- [Build and test development](#build-and-test-development)
- [Compare changed columns](#compare-changed-columns)
- [Explain differences by cause](#explain-differences-by-cause)
- [Accept or escalate differences](#accept-or-escalate-differences)
- [Downstream validation is opt-in](#downstream-validation-is-opt-in)
- [Apply the quality gate](#apply-the-quality-gate)
- [Record evidence](#record-evidence)

## Set the validation scope

Select a mode before warehouse execution; user scope and constraints take
precedence. Full means completing that mode, never following the DAG by default.

| Mode | Required scope |
| --- | --- |
| Behavior-preserving reporting refactor | Every row and comparable output column in the directly changed models over the agreed window; separately check added/removed columns and keys. |
| Focused feature or bug fix | Added, removed, renamed, retyped or logic-changed outputs plus minimal comparison keys/invariants. All outputs of a new model are new. |
| Documentation, comments or descriptions | Syntax, reference and semantic consistency; recheck affected quoted values or example queries. No automatic model materialization or all-column data run. |

An explicit request for row-by-row identity or all columns selects full-row
coverage for the named models. Do not reduce that to changed columns. Conversely,
formatting changes alone do not justify full-row validation. Record the mode and
model-to-column scope in the existing handoff; extend it for shared macros or
ephemeral outputs only as needed. Downstream data checks remain opt-in in every
mode; a model-level request does not silently select its descendants.

For substantial reporting refactors without a requested period, use trailing
12 months through yesterday as an overridable starting scope, subject to source
availability and execution constraints. Honor an agreed window across follow-ups.
Under no-refresh instructions, cap comparisons at the common retained freshness
cutoff and label partial days or weekly/monthly buckets. If that scope is not
feasible, report the concrete limit rather than silently shortening the period.

Reporting dependency deprecations additionally require a live content-impact
check under [reporting-impact.md](reporting-impact.md), even when the changed
column allowlist is empty. This is separate from downstream data validation.

## Protect the comparison

- Keep production read-only during development validation.
- Use a task-specific schema and separate production-state, target, and log
  paths. Confirm resolved relations before warehouse writes.
- Tie the mode, column scope and results to the current commit and working-tree
  fingerprint. Re-evaluate the allowlist after a correction.
- Use the same time and business predicates on development and production.
  Record production baseline timestamps and any source-freshness differences.
- Identify the baseline as actual production output/snapshot, production logic
  evaluated on matched inputs, or an intermediate development result. Only the
  first is a direct production-output comparison. Use matched-input replay to
  isolate causes when needed; do not substitute it for end-to-end evidence.
- Document normalization, tolerances and nondeterministic-column exclusions.
  Preserve exact NULL/zero differences, array-order versus membership differences,
  and financial cent-rounding effects separately from aggregation noise.
- A no-refresh instruction forbids refreshing parents, cloning new data or
  rebuilding warehouse tables. Use retained relations or read-only candidate
  SELECTs against them; record freshness, executable provenance and method. A
  date cutoff alone does not remove historical source revisions.

## Build and test development

Choose checks appropriate to the mode. Documentation-only checks can reuse
applicable data evidence after executable/contract equivalence is recorded.
For executable model changes, the normal path below applies; use read-only
inline execution when no refresh/build is authorized. Never claim a materialized
schema check or a dbt test run from a SELECT-only comparison.

1. Parse the project, compile directly changed models, and lint modified files.
   Record parse and compile outcomes separately; one does not prove the other.
2. Clone or defer unchanged production parents needed to build those models.
   Dependency preparation does not add those parents to the validation scope.
3. Materialize only the named models in the isolated environment. Do not use a
   trailing `+`, descendant selector, or full-project build by default.
4. Select test nodes whose assertions target the mode's selected columns,
   plus any minimal key checks required for comparison. Inspect
   generic, singular, relationship, and unit tests for unrelated assertions.
   Do not run a whole-model suite just because it is attached to that model.
5. Verify selection with `dbt ls` or the manifest before execution. Avoid eager
   indirect selection that pulls in out-of-scope columns or downstream tests.

Prefer `dbt run` for the named models followed by `dbt test` with an explicit
list of relevant test nodes and `--indirect-selection empty` when supported by
the project's version. A scoped `dbt build` is also acceptable if its resolved
model and test list stays within the allowlist. If no relevant test nodes
exist, use focused comparison queries; never fall back to the entire suite.
Keep existing tests in the repository; select their execution according to the
recorded mode and columns. See dbt's
[indirect selection guidance](https://docs.getdbt.com/reference/global-configs/indirect-selection)
when resolving test selection for the installed version.

Stop after required checks pass; repeat or broaden only for a code change,
failure, unresolved concern or explicit expanded request.

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
Matching a subset of columns does not establish whole-row parity. Full-row
identity requires all comparable columns and key differences to be assessed.

For each selected comparable column, report changed-value counts and rates,
null-rate and distinctness changes, safe example keys, and whether differences
match the ticket. Add meaningful numeric deltas, category frequencies, date
boundaries, or other type-specific checks only for selected columns. Project
only keys and selected values into comparison queries; avoid `select *` and
whole-row hashes that conceal which columns changed.

Profile added columns against their declared invariants. Use complete
in-scope populations for exact comparisons; disclose sampling or approximate
metrics rather than describing them as exhaustive evidence. Missing required
in-scope evidence prevents a pass. A documentation-only change may need
only static/reference checks. For a reporting refactor, unchanged formulas are
precisely what the selected all-column comparison must verify.

## Explain differences by cause

For material data differences, prepare a cause-based table for the PR's
validation section. Compare the tested development result with the actual
production output over the declared population, window, and selected columns.
An earlier development iteration is not a production baseline. After a repair,
recompute production comparisons rather than relabeling iteration-to-iteration
counts. For a new model without a comparable production output, state that
limit and report its applicable invariant checks; do not invent differences.

Use these fields, with names adapted to the model and business keys:

| Field | Required evidence |
| --- | --- |
| Change | The demonstrated cause of a distinct class of differences, rather than just "values changed." |
| Description | How the selected output differs between production and development, including null transitions, added/removed records, or changed metric definitions. |
| Accepted? | Accepted, rejected, or pending, with the reason and acceptance source under the rules below. |
| Impact (unit) | A measured quantity with its unit, aggregation, and relevant scope explicitly defined. |
| Example record key | A real, safe identifier belonging to that exact difference category. |
| Example related entity key | The related order, customer, user, or other parent key from the same example record, when meaningful. |

Choose impact measures that answer the business question at the declared
grain. Examples include affected model rows, `count(distinct customer_id)`,
`count(distinct user_id)`, distinct orders, or a monetary/quantity delta. Count
stable comparison keys instead of multiplied join rows. Derive impact from
the in-scope comparison; this format does not authorize unrelated column or
downstream validation. For material measure changes, also retain production,
development, and absolute/relative delta with units and window. A count of
affected entities is not a substitute for the value of a material metric delta.

When showing a changed-record total, classify each comparison key once using
documented precedence or combine overlapping memberships before counting.
Distinct customers/users can occur in multiple categories even when records
do not: label that overlap and calculate the overall distinct count from the
union, not the sum of per-category counts. Keep added/removed keys and freshness
differences identifiable. Do not mix units or sum unrelated metrics into a
single total. State the denominator for percentages and disclose sampling or
approximation.

Verify example identifiers and their related keys from the same row; do not
pair a line ID with an order ID from another example. For aggregate models,
use the actual grouping keys and reporting period rather than inventing a row ID. Keep exact category
queries, memberships, metric definitions, and acceptance sources in the handoff
so the deployer can publish the table without reconstructing the investigation.
If a category has no safe record-level example, explain why instead of
inventing one. Zero-difference or static-only validation can be summarized
without a fabricated change table.

For material cohort changes, reconcile entering/leaving IDs, retained IDs that
move date or attribution groups, and any duplicate contribution across old
buckets. Distinguish a population total from a net increase: unlinked orders in
development are not automatically newly unlinked orders. Separate raw counts,
allocated credits and additional cohorts before claiming parity. Include a safe
actual grouping-key example and the production/development values. Explain
large row-level revenue/CAC/CVR/AOV changes even when period totals barely move.

## Accept or escalate differences

Classify each difference by cause and acceptance, separately from magnitude:
approved behavior change, demonstrated representation/precision effect,
source/environment discrepancy, or unexplained difference. An operational
explanation does not itself authorize changed reporting values.
Keep exact counts, deltas, comparison windows, run timestamps, and supporting
queries internally even when the PR only needs a short explanation.

- Precision differences can be accepted when demonstrated by unchanged-logic
  evidence or source arithmetic and within a justified tolerance. Do not
  choose a tolerance after seeing results merely to hide a defect.
- Differences caused solely by model run times (for example, development has
  fresher data) are generally acceptable when timestamps and aligned-window
  checks or row evidence demonstrate the cause. Do not use "freshness" to
  excuse unexplained differences in overlapping history.
- Unchanged SQL alone does not establish cause; verify timestamp types, session
  settings and captured parents before attributing differences to timezone or
  freshness. Keep unverified causes unresolved.
- Logic changes that affect metrics require explicit user acceptance of the
  specific behavior and scope, in the ticket or conversation. The developer's
  explanation, small magnitude, or consistency with its implementation is not
  approval. A broad request to refactor is not acceptance of metric drift.
- When the cause, materiality, or acceptance is uncertain, ask the user through
  the coordinator with the metric, production/development values, scope,
  proposed cause, and decision needed. Standalone testers ask directly.
  Continue independent checks, but do not issue a pass for the unresolved
  scope. A known defect is `FAIL`; a pending acceptance decision is `BLOCKED`.

In the table, distinguish a verified correction from accepted unresolved
handling. An accepted null or excluded association can mean the evidence is
insufficient; it does not prove the record belongs to another business class
or that historical attribution is complete. Show pending decisions honestly
rather than treating implementation intent or passing tests as acceptance.
Honor acceptance already provided for the behavior and scope; do not ask again.

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

- `PASS`: all required checks for the recorded mode/scope ran, selected-column
  differences are explained and accepted under the rules above, comparison
  keys are usable, and evidence matches
  the current fingerprint. For reporting deprecations, the required live
  impact check must also be resolved under `reporting-impact.md`.
- `FAIL`: a correctable implementation or test defect exists within that scope.
- `BLOCKED`: access, missing data, or unresolved semantics prevent required
  in-scope validation. Never block solely because columns outside the recorded
  mode or unrequested downstream resources were not tested.

After a fix, rerun affected checks under the recorded mode and column scope;
do not broaden validation beyond the recorded mode or named models.

## Record evidence

Include the model/column allowlist, exact relations, key, comparison predicates,
commands, results, safe examples, and artifact paths in the tester packet from
[agent-handoffs.md](agent-handoffs.md). Report columns outside the selected mode as excluded and downstream validation
as not requested unless explicitly included. Keep
post-deployment checks within the same scope unless the user expands it.
Record reporting-deprecation impact separately, including live coverage and
limitations. Keep commands, JSON, fingerprints, and detailed inventories in
the packet; the PR receives a concise findings summary, not the raw packet.
Record parse, compile, run, and test statuses separately, with their scopes,
failure/warning/skip counts, and acceptance sources for differences. Missing
tests or a compile-only check cannot be summarized as a successful test run.

Preserve isolated schemas under the user's retention decision. Ask about cleanup
only if no decision exists; retained schemas are not an outstanding blocker.
