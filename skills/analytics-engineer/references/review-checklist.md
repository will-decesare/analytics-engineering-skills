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

- Is the production contract recorded, including required non-changes and approved
  exceptions? Does the refactor preserve population, names, output coverage and
  allocation unless a specific change is accepted?
- Do timestamp types, upstream conversions and session settings explain date
  behavior? Is changing the event date kept separate from changing timezone?

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
- Can redundant CTEs, joins, dependencies, or diagnostic fields be removed?
- Are remaining CTEs used and clear, with final projection mostly assembly?
- Does the SQL comply with local dialect and lint rules?
- Does each test catch an independently plausible contract failure rather than
  repeat its implementation? Remove obsolete coverage and documentation within
  scope; retain meaningful invariants and defer unrelated cleanup.

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

- Does the evidence satisfy [full-validation.md](full-validation.md) for the
  requested mode, columns, baseline, and deployment scenarios?
- Were comparison scope and relation preparation separated, preserving archives
  and deprecated inputs while rebuilding the required active dependency path?
- Do tested fixtures reproduce the production history that deployment will read?
- After integrating child PRs or base changes, was affected evidence renewed
  for the combined state rather than combining unrelated passes?
- Are metric definitions, periods, prose, tables, and exports consistent, with
  causes reconciled and snapshot/release differences distinguished from PR logic?
- Were explicit stop/review, no-refresh, retry, and warehouse constraints honored?
- Are unrun checks disclosed and acceptance decisions supported, including the
  distinction between unresolved associations and verified business corrections?

## PR preparation

- Does the final description follow [pull-request-workflow.md](pull-request-workflow.md):
  sourced motivation, current scoped evidence, required rendered
  DAG images, accepted differences, and actionable release work?
- Were superseded alternatives, investigation narrative, and explanations of
  unchanged behavior removed while preserving reviewer edits and limitations?
- Are local edits accessible on the intended named branch, with the absolute
  worktree path reported? Were untouched formatting and explicit projections
  preserved rather than changing them merely to reduce line count?

## Report findings

- Before expanding implementation, classify findings as introduced, inherited,
  or exposed by the change. Quantify observed affected records and business
  impact where feasible; label zero observed impact, unknown coverage, and
  hypothetical failure scenarios distinctly. Zero observed impact alone neither
  proves safety nor justifies speculative fallback logic.
- Give severity, precise file/line, violated invariant, evidence, and disposition:
  fix now, defer, or dismiss, with rationale and any required acceptance.
- Recommend the smallest correction; keep unrelated preventive work separate.
- If no actionable issues remain, say so and identify any residual validation
  gaps.
