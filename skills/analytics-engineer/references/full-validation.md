# Full Development-versus-Production Validation

## Contents

- [Protect the comparison](#protect-the-comparison)
- [Resolve the affected graph](#resolve-the-affected-graph)
- [Build and test development](#build-and-test-development)
- [Compare relation schemas](#compare-relation-schemas)
- [Compare rows and column values](#compare-rows-and-column-values)
- [Validate downstream models](#validate-downstream-models)
- [Apply the quality gate](#apply-the-quality-gate)
- [Record evidence](#record-evidence)

## Protect the comparison

- Keep production read-only and use the repository's supported production
  target only to resolve state or execute safe comparison queries.
- Use a collision-resistant task schema and separate target, log, and
  production-state paths. Confirm resolved relation names before every
  warehouse-writing command.
- Capture the commit SHA and a fingerprint of staged and unstaged changes before
  validation. Restart if that fingerprint changes.
- Capture the production relation names and baseline timestamp. Apply identical
  time, status, tenant, and business predicates to development and production.
- Identify audit columns whose values are intentionally nondeterministic. Do not
  silently exclude them; document the normalization or exclusion.

## Resolve the affected graph

Use the dbt manifest or repository graph tools to enumerate:

1. every changed model;
2. parents required to build those models safely;
3. every direct and indirect materialized descendant whose rows or contract can
   change;
4. tests, snapshots, semantic models, metrics, exposures, and BI contracts tied
   to those resources.

Record an explicit selector or model list before building. Inspect broad graph
selectors so they do not unexpectedly include unrelated or production-bound
resources. Ephemeral nodes do not need relation comparisons, but their effects
must be tested through materialized descendants.

Full validation requires every materially affected descendant. If credentials,
cost limits, unavailable fixtures, or an unbounded graph prevent this, return
`BLOCKED` unless the user explicitly changes the required scope. Do not label a
partial descendant sample as full validation.

## Build and test development

Follow repository commands and versions. A typical ladder is:

1. parse or compile the changed selector;
2. lint changed SQL and YAML;
3. clone or defer unchanged production parents into the task environment;
4. build changed models and affected descendants in development;
5. run schema, relationship, singular, unit, and business-invariant tests;
6. confirm every created relation is in the task namespace;
7. reconcile model grain, key uniqueness, row counts, and additive measures.

Use targeted selectors, but do not omit a descendant merely to shorten a full
validation run. Record each exact command, outcome, runtime, and artifact path.

## Compare relation schemas

For each changed model and affected materialized descendant, compare the
development and production contracts. Report:

- development-only and production-only columns;
- column order changes when order is a consumer contract;
- data type, precision, scale, length, nullability, and casing changes exposed
  by the adapter;
- description, test, semantic, metric, exposure, and BI definition changes;
- whether each difference is required by the ticket and backward compatible.

For a new model with no production counterpart, validate the declared schema,
grain, types, descriptions, tests, and consumer contract without fabricating a
production comparison.

## Compare rows and column values

Use the declared stable grain key. For composite grains, compare the complete
key. Verify key uniqueness independently on both sides before trusting a join.

Use a full outer comparison with null-safe equality to classify:

- development-only keys;
- production-only keys;
- matched keys with at least one changed comparable column;
- exact matching keys.

For every comparable shared column, report at minimum:

- matched rows whose value changed and percent of matched rows;
- development and production null counts and null rates;
- development and production distinct counts or a documented approximation;
- safe representative changed keys;
- whether the change is expected, unexpected, or still unexplained.

Add type-appropriate profiles where meaningful:

- numeric: minimum, maximum, sum, average, total delta, and row-level absolute
  or relative deltas using a justified tolerance;
- categorical and boolean: value-frequency changes, including null;
- date and timestamp: minimum, maximum, timezone normalization, and boundary
  changes;
- strings: trimmed or normalized comparison only when the contract permits it,
  plus length and blank-value changes;
- arrays, objects, or variants: deterministic canonicalization before equality
  when supported.

Profile new columns separately for population rate, distinctness, ranges,
accepted values, and ticket-specific invariants. Do not treat matching row
counts or measure totals as proof of column-level parity.

Use complete in-scope relations when feasible. If sampling or approximate
statistics are necessary, disclose them and return `BLOCKED` for a workflow
that requires full validation unless the user accepts the reduced assurance.

## Validate downstream models

Repeat the schema, grain-key, row classification, and per-column value analysis
for every materially affected materialized descendant. A downstream model can
change even when its schema does not, so value comparisons are mandatory.

Additionally verify:

- row preservation or intentional row changes at each descendant grain;
- metric numerator, denominator, eligibility, and time-window behavior;
- aggregate measure reconciliation from the changed parent to descendants;
- renamed, removed, retyped, or newly populated fields in semantic and BI
  contracts;
- representative ticket edge cases through the complete lineage.

Distinguish propagation of an expected upstream change from a downstream
regression. Explain material differences model by model and column by column.

## Apply the quality gate

Return `PASS` only when:

- all required commands and tests succeeded;
- changed models and all materially affected descendants were validated;
- grain keys are valid and no unintended fan-out or row loss exists;
- every material schema and per-column value difference is explained by the
  ticket or approved business behavior;
- no unexplained contract, metric, or BI regression remains;
- evidence is tied to the current code fingerprint.

Return `FAIL` for a correctable implementation, test, or contract defect. Return
`BLOCKED` when required validation cannot run or acceptance criteria cannot
resolve whether a difference is correct. Never weaken thresholds after seeing a
failure unless the business rule justifies the change.

## Record evidence

Produce compact model-level and column-level tables. Include exact relations,
scope, counts, percentage changes, safe example keys, commands, log paths, and
limitations. Redact sensitive data and link to access-controlled artifacts when
row samples cannot be posted safely.

Follow the Tester Pass Packet or Tester Failure Packet in
[agent-handoffs.md](agent-handoffs.md). Preserve the isolated schema until the
deployer has created the PR and the user has answered the cleanup question.
