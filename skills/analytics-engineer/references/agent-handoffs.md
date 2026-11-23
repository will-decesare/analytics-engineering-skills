# Analytics Agent Handoffs

Use structured packets so each role acts on the same ticket and code state. All
routine packets return to the analytics-engineer coordinator, which owns the
workflow state and routes the next specialist. Do not make the user relay
routine status between roles.

These are internal working packets, not PR-description templates. Retain exact
commands, raw/API results, artifact paths, and fingerprint verification here
for reproducibility. Do not paste them into PR descriptions or comments;
publish only the concise summary defined in `pull-request-workflow.md` unless
the user requests technical detail or a repository template requires it.
The deployer incorporates the concise summary into the original PR description
and updates it in place. No role posts a separate validation comment unless
the user explicitly requests one.

## Shared identity

Every packet must include:

- workflow or ticket identifier and link when available;
- iteration number;
- repository, named branch, absolute worktree path, and intended base branch;
- current commit SHA plus a fingerprint of staged and unstaged changes;
- isolated schema names and dbt artifact/log paths;
- work type, validation mode and comparison model/column allowlist; separately,
  the relation plan (rebuild/clone/preserve/defer) and reasons for inclusions;
- production contract, required non-changes and specifically approved exceptions;
- comparison record under [full-validation.md](full-validation.md#protect-the-comparison),
  including baseline kind, exact relations/code/inputs, freshness, reporting
  window, grain, metric formulas, and execution method;
- explicit downstream data-validation request and its model/column scope, or
  `not requested; not performed`;
- reporting-deprecation impact scope and check status, or `not applicable`,
  separate from optional downstream data validation;
- authority already granted for commits, pushes, PR creation/description
  updates, explicitly requested comments, and cleanup;
- execution mode (normal delivery, diagnostic single attempt, local review, or
  read-only), allowed mutations/refreshes, stop/review conditions and retry scope;
- completed stages and evidence, next/resume point, and warehouse resize limits
  and required restoration; preserve these across handoffs and follow-ups;
- packet author role and timestamp;
- requested model tier, resolved model ID and supported reasoning effort,
  selection basis, actual settings when exposed (otherwise `not exposed`),
  and any model fallback or escalation with its reason.

Update this state in place. Executable/contract changes renew affected checks;
substantial merges require reassessing the integrated diff and deployment path.
For prose-only changes, record unchanged executable definitions, any rechecked
recipes/values, and evidence still applicable. Do not relabel old output as a
new run or add another handoff layer.

For delegated routine evidence, include the worker's assignment, fingerprint,
exact sources/commands, timestamps, scope and coverage, results, artifact paths,
and unresolved issues. Return it through the coordinator to the requesting
specialist under [model-routing.md](model-routing.md). This is supporting
evidence, not a Tester Pass Packet; the tester retains the final verdict.

## Developer Handoff Packet

Include:

- ticket acceptance criteria and exclusions;
- motivation and its ticket/user source, or the pending motivation question;
- explicitly user-accepted logic/metric changes and the behavior/scope covered;
- branch preflight: authoritative remote/base, successful fetch time, local
  ahead/behind result, update or isolation action, and verified starting tip;
  when resuming an existing branch, preserve its original branch-point record
  rather than claiming a new preflight occurred;
- changed SQL, YAML, tests, docs, semantic resources, and BI contracts;
- purpose, grain, stable key, materialization, and refresh behavior per model;
- source ownership, join cardinalities, cutoffs, and null behavior;
- tests added or changed and why they protect the contract;
- expected row, column, measure, history, and downstream changes;
- any downstream impact noted from lineage, kept separate from the validation
  allowlist; an impact note does not authorize downstream execution;
- retiring reporting dependencies, exact BI identifiers/mappings, known live
  consumers, and proposed replacements for the required API impact check;
- developer commands and outcomes;
- review dispositions under [review-checklist.md](review-checklist.md#report-findings),
  known risks, unavailable checks, and post-merge work;
- for DAG changes, the before/base revision or preserved docs location and the
  affected nodes to capture under [dag-screenshots.md](dag-screenshots.md).

Address it to the analytics-engineer coordinator with the explicit state
`READY_FOR_TEST`. The coordinator verifies it before starting the tester.

## Tester Failure Packet

Include:

- status `FAIL` and tested fingerprint;
- failing command, model, relation, grain key, column, or invariant;
- comparison scope and isolated development relation;
- expected and actual results;
- row and per-column difference counts plus safe representative keys;
- in-scope effect and severity; downstream data results only when requested;
- for reporting deprecations, live impacted assets and unresolved breakage;
- relevant log or artifact path;
- failure class (runner/environment, stale dependency, implementation, or
  unresolved acceptance), smallest correction, and last completed stage;
  do not edit implementation or advance a stopped sequence.

Return to the coordinator, which applies the recorded repair/review conditions.
An authorized repair needs a new fingerprint and Developer Handoff Packet.

## Tester Pass Packet

Include:

- status `PASS` and tested fingerprint;
- exact parse, compile, lint, clone/defer, run/build, and test commands and
  separate outcomes, model scopes, and failure/warning/skip counts;
- applicable deployment scenarios, results and unverified paths, including
  retained archive inputs and incremental write-bound checks when relevant;
- row keys and development-only, production-only, changed, and matching rows,
  with matching defined only over the selected comparable columns;
- schema checks for selected added, removed, renamed, or retyped columns;
- selected-column value comparisons and type-specific profile deltas;
- differences with demonstrated causes and acceptance status/source under
  [full-validation.md](full-validation.md#accept-or-escalate-differences);
- reviewer-ready production-to-development cause tables under
  [full-validation.md](full-validation.md#explain-differences-by-cause): Change,
  Description, Accepted? with rationale/source, Impact with its unit and
  aggregation, a verified example record ID, and paired related-entity IDs;
- category membership queries, overlap/precedence rules, denominators, and
  reconciled totals (including union-based distinct-entity totals when needed);
- material key-metric production/development values, deltas, and reasons;
  summarize negligible accepted measure drift without publishing exact deltas,
  while preserving those values internally;
- new-model checks where no production relation exists;
- downstream data-validation results only when explicitly requested; otherwise
  `not requested; not performed`, not an outstanding validation gap;
- for reporting deprecations, the required live API check: timestamp, coverage,
  environment/candidate tested, impacted asset names/IDs/links, required
  migrations or accepted retirement decisions, and any visibility limits;
- limitations, cleanup candidates, and post-merge production checks.
- for dependency changes, before/after dbt Docs DAG screenshots, code/artifact
  identities, capture paths, and image-inspection results; record missing
  screenshots separately from the data verdict.

Missing API access or insufficient impact coverage belongs in a blocked
report, not a pass packet claiming that reporting content is unaffected. See
[reporting-impact.md](reporting-impact.md) for the deprecation gate.
Likewise, an uncertain or unapproved logic-driven metric change belongs in a
blocked report with a precise question for the user; small size does not imply
acceptance. Explicit downstream compile/run/test coverage must be distinguished
from downstream value comparisons, which require their own requested scope.

Address it to the analytics-engineer coordinator and attach the latest
Developer Handoff Packet. The coordinator verifies the fingerprint before
starting the deployer.

## Deployer Completion Packet

Include:

- current code fingerprint and matching tester pass fingerprint;
- commit and push actions actually performed;
- PR title, URL, base, head, and draft or ready state;
- PR-description source, updated validation section, and confirmation that the
  latest body was read back and unrelated content preserved; a comment URL is
  neither required nor expected unless the user explicitly requested a comment;
- concise validation and reporting-impact summary suitable for the user,
  without commands, JSON inventories, local paths, or hash/provenance details;
- confirmation that the description includes the sourced motivation and that
  validation starts with dbt outcomes before comparison tables and reasons;
- for DAG changes, durable Before/After image URLs and confirmation both
  dropdowns render in the PR, or the precise screenshot-preparation blocker;
- CI or dbt job state, never implying pending checks passed;
- remaining pre-merge and post-merge work;
- release coordination and exceptional post-merge actions for the PR, excluding
  routine smoke-test and production-verification reminders;
- exact isolated schemas still present;
- whether retaining those schemas supports review or additional validation;
- the recorded schema-retention decision, or whether an unanswered cleanup
  question is needed.

Address it to the analytics-engineer coordinator. The coordinator tells the
user that the PR is ready for review only when the PR actually exists, the body
has been verified, and all actions required by the authorized scope are
complete.

Role boundaries and routing are defined in [SKILL.md](../SKILL.md#preserve-role-boundaries).
Report missing evidence or stale state rather than filling packet fields with
assumptions.
