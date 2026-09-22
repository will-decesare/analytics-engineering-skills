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
- For reporting deprecations, did live API checks identify saved-content
  impact, with coverage limits and migration decisions recorded? Repository
  searches alone do not satisfy [reporting-impact.md](reporting-impact.md).
- Are task, release, or backfill instructions updated if needed?

## Validation evidence

- Are parse, compile, run, and test outcomes and actual scopes reported first,
  with failures, warnings, and skips distinguished? Did applicable linting run?
- Were tests selected only for changed columns in directly changed models,
  with any key checks limited to establishing a valid comparison?
- Were development and production validation reported separately?
- Was development compared with production at the declared grain and over the
  same scope, with development-only, production-only, and changed rows
  classified using selected-column equality rather than whole-row parity?
- Were null rates, measure deltas, and edge cases checked only for the selected
  columns, with unchanged-column profiling excluded?
- Was downstream model and column validation performed only when explicitly
  requested, and otherwise marked outside scope without blocking the pass?
- Are unavailable checks disclosed instead of implied successful?
- Are changed-column results and large key-metric differences shown in tables,
  including causes and acceptance status? Are exact negligible accepted deltas
  kept internally with a concise reason in the PR?
- Are rounding/freshness explanations demonstrated? Did the user explicitly
  accept logic-driven metric changes, and were uncertain decisions escalated
  instead of treating a plausible explanation as approval?
- Are concise validation findings maintained in the original PR description,
  with no separate validation comment unless explicitly requested?
- Were unrelated PR-description content and reviewer edits preserved, with
  commands, JSON, local paths, and provenance details kept in internal evidence?

## PR preparation

- Does Description and Motivation explain why the change is needed using the
  ticket or user response? Was missing motivation asked for rather than invented?
- Does the pre-merge list contain only unresolved release coordination, with
  actual validation blockers kept visible in the validation section?
- For DAG changes, were both real dbt Docs sites generated and served, their
  DAGs captured and inspected, and the Before/After dropdown images verified
  in the rendered PR?
- Are post-merge tasks limited to exceptional actions, with routine smoke
  testing and production-verification reminders omitted?

## Report findings

- Report only actionable findings caused or exposed by the change.
- Order findings by severity and include a precise file and line location.
- State the violated invariant, a concrete failure scenario, and the likely
  impact.
- Recommend the smallest safe correction without expanding the user's scope.
- If no actionable issues remain, say so and identify any residual validation
  gaps.
