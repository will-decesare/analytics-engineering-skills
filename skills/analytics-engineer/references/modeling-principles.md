# Modeling Principles

## Begin with grain

- Define what one row represents before selecting fields.
- Choose a stable key that proves that grain; distinguish a business key from a
  warehouse surrogate key.
- Make measures additive only across dimensions where addition is valid.
- Treat a grain change as an interface change, even when column names remain.

## Control join cardinality

- Classify every join as one-to-one, many-to-one, one-to-many, or many-to-many.
- Make the intended row-preserving side explicit.
- Aggregate, deduplicate by a documented rule, or bridge the many side before a
  join that should preserve the target grain.
- Validate uniqueness on join keys and compare pre/post-join row counts and
  measure totals.
- Never use `distinct` as an unexplained repair for a fan-out.
- Normalize key types, case, whitespace, and null sentinels in upstream steps,
  not inside join predicates.

## Place logic by ownership

- Keep source-shaped renaming and cleanup near ingestion.
- Put shared identity resolution and reusable business concepts upstream of
  their consumers.
- Put entity-level attributes and measures on the cohesive entity model that
  owns them when that reduces duplicated joins and DAG branches.
- Keep report-specific filters, labels, and layouts near the reporting layer.
- Avoid both premature abstraction and repeated business rules. Add an
  intermediate model only when it creates a stable, reused contract.

## Handle duplicates deliberately

- Determine whether duplicates are ingestion defects, source revisions,
  legitimate repeated events, or grain mismatches.
- Repair ingestion defects at the earliest controllable source layer.
- Use a window function only with a business-backed ordering rule and a stable
  tie-breaker.
- Preserve all records when multiple rows are legitimate; change the grain or
  aggregate instead of silently choosing one.

## Model time and migrations explicitly

- Identify event time, processing time, effective time, and reporting date.
- Declare timezone conversion and date-boundary behavior.
- For system migrations, configure an authoritative cutoff and use mutually
  exclusive intervals so records cannot be omitted or double counted.
- Define the source of truth for each era and normalize cross-system entity IDs
  before combining facts.
- Test the boundary dates, overlap/gap conditions, and representative entities
  that exist in both systems.
- Document how late-arriving records and backfills cross the cutoff.

## Design incremental models for replay

- Make the unique key match the model grain.
- Choose an incremental predicate that captures late updates, not only newly
  created rows.
- Define lookback, merge, deletion, schema-change, and full-refresh behavior.
- For cumulative or semi-additive snapshots, recompute from the earliest
  affected event through every later derived state; overwriting only the fixed
  lookback partitions may leave future rows stale.
- Keep results idempotent: rerunning the same input should produce the same
  output.
- Compare incremental output with a bounded full recomputation when risk is
  material.

## Preserve metric meaning

- Define numerator, denominator, eligibility population, filters, and time
  window for each metric.
- Separate event facts from snapshot facts and semi-additive balances.
- Avoid summing pre-aggregated values across incompatible grains.
- Centralize shared metric definitions in the repository's semantic or metric
  layer when one exists.

## Reduce operational cost safely

- Filter and project early when semantics allow.
- Avoid scanning the same large relation multiple times.
- Prefer a smaller coherent dependency graph over convenience models that fan
  out and repeat work.
- Optimize after correctness and measure the impact using query plans, scan
  volume, runtime, or warehouse cost.
