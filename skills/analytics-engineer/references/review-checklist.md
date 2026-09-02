# Analytics Engineer Review Checklist

Prioritize correctness and contract regressions over formatting. Review the
diff in the context of parent models, consumers, metadata, and local rules.

## Grain and joins

- Does each changed model have a clear row grain and valid unique key?
- Can any join multiply or silently drop rows?
- Is the many side reduced before a grain-preserving join?
- Are key types and normalizations prepared upstream?
- Is `distinct` or window deduplication masking a source or join defect?

## Business and temporal semantics

- Do calculations match documented business definitions?
- Are numerator, denominator, eligibility, nulls, and zero behavior correct?
- Are date, timezone, and inclusive/exclusive boundaries explicit?
- Do migrations or source cutovers avoid both overlap and gaps?
- Is cross-system identity resolved before aggregation?

## Incremental and operational behavior

- Does the unique key match the grain?
- Are late updates, deletions, lookback, and full refresh handled?
- Can a late correction change cumulative or snapshot rows beyond the rewritten
  incremental window?
- Is the transformation idempotent?
- Could the change cause an unnecessary large scan or DAG fan-out?
- Does logic now exist in more than one place?

## SQL and architecture

- Does the logic live at the earliest stable semantic owner?
- Was moved logic integrated coherently rather than appended?
- Are CTEs single-purpose, used, and named clearly?
- Is the final step mostly assembly and projection?
- Does the SQL comply with local dialect and lint rules?

## Contract completeness

- Are new or changed models and columns documented and tested?
- Are semantic models, metrics, exposures, selectors, and BI definitions synced?
- Are renamed or removed fields treated as breaking contracts?
- When a model is retired, is all active metadata and lineage removed?
- Are task, release, or backfill instructions updated if needed?

## Validation evidence

- Did parsing/compilation and linting run?
- Were targeted model and data tests executed?
- Were development and production validation reported separately?
- Was development compared with production at the declared grain and over the
  same scope, with development-only, production-only, and changed rows
  classified?
- Were uniqueness, nulls, row counts, measure totals, and representative edge
  cases checked?
- Were downstream consumers validated when the interface changed?
- Are unavailable checks disclosed instead of implied successful?

## Report findings

- Report only actionable findings caused or exposed by the change.
- Order findings by severity and include a precise file and line location.
- State the violated invariant, a concrete failure scenario, and the likely
  impact.
- Recommend the smallest safe correction without expanding the user's scope.
- If no actionable issues remain, say so and identify any residual validation
  gaps.
