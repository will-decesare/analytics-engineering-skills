---
name: analytics-engineer
description: Coordinate analytics work tickets through development, contract-aware validation, repair loops, and a concise evidence-backed PR. Use as the default analytics workflow entry point. Downstream data validation is opt-in; reporting deprecations require live content-impact checks.
---

# Analytics Engineering Workflow

Coordinate three independent roles while preserving repository rules, test
integrity, and explicit authority for external actions.

## Route to the correct role

- Use `$analytics-engineer` as the default entry point for a new work ticket and
  the owner of the end-to-end workflow.
- Use `$analytics-developer` directly only for an implementation-only request
  or to resume a developer phase outside a coordinated workflow.
- Use `$analytics-tester` to validate an existing developer handoff against
  production, using the validation mode for the directly changed models. Include downstream data validation only when explicitly requested;
  require live reporting-impact checks for deprecations as described below.
- Use `$analytics-deployer` only after a current tester pass to prepare the PR
  description with its validation summary and the authorized draft pull
  request, or update that same description when the PR already exists.

Own the ticket, code and environment identity, execution constraints, validation
status, and next role in the existing [handoffs](references/agent-handoffs.md).
Carry decisions and completed stages across agents and follow-ups; the user
should not have to reconstruct state or relay routine handoffs.

## Preserve the reporting contract

For source refactors, preserve production populations, output keys, field names,
metric meaning, attribution coverage, allocation and timezone behavior unless
specific changes are requested. Record the contract and approved exceptions
under [modeling-principles.md](references/modeling-principles.md#record-the-production-contract).
A source switch alone does not authorize broader reporting populations or new
metrics. Carry the user's decisions forward across repairs and follow-ups.

Choose the validation mode under
[full-validation.md](references/full-validation.md#set-the-validation-scope).
Independent development/testing remains the path for executable model changes.
For documentation, comments, descriptions or PR-text follow-ups, use the relevant
specialist directly and reuse applicable evidence after confirming executable
logic and contracts are unchanged. Recheck affected reference queries when their
recipes or quoted values change; do not launch a warehouse rebuild for prose.

## Select models for the work

Read [model-routing.md](references/model-routing.md) before starting agents.
Use its strong defaults for planning, development, and the independent tester,
and its cheaper default for deployment. For bounded routine evidence collection
requested by a specialist, launch an evidence worker with the shared default
and route its evidence back to that specialist. The strong tester retains test
design, interpretation, and the final gate. Apply the reference's launch,
fallback, and escalation rules; do not rely on default model inheritance to implement this routing.

## Run the workflow

At intake, capture both the requested change and its motivation from the ticket
or the user's explanation. If the reason is missing, ask the user for it early
while continuing independent investigation. Do not invent a rationale or treat
an implementation description as motivation. Carry the answer into the
developer handoff and PR's Description and Motivation section.

1. Read the ticket and repository instructions; establish scope, execution mode,
   stop conditions, and authority. Start a separate `$analytics-developer` using
   the [branch guidance](references/dbt-workflow.md#start-from-current-remote-main):
   reuse the intended PR checkout, or verify current remote code for a new branch.
2. Receive the Developer Handoff Packet, verify its code fingerprint and
   `READY_FOR_TEST` state, and record the comparison scope separately from the
   relation preparation plan before starting a separate `$analytics-tester`.
3. Have the tester validate the isolated development state against production
   under [full-validation.md](references/full-validation.md), including applicable
   deployment scenarios within the authorized scope. Preserve the user's
   no-refresh, single-attempt, stop/review, and warehouse constraints.
4. Require schema and row-keyed value comparisons for the selected columns.
   Reporting refactors default to all-column comparisons in directly changed
   models; focused fixes use changed outputs. Keep unrelated tests out of scope.
   For uncertain acceptance or logic-driven metric changes not explicitly
   accepted by the user, ask with the observed impact and proposed explanation.
   Continue independent work, but keep the affected validation gate unresolved.
5. On `FAIL`, receive the Tester Failure Packet and route it to the developer.
   Repair and retest only when the execution mode permits; otherwise report the
   cause and recommendation and await review. Resume from verified completed
   stages when authorized, renewing only affected evidence.
6. On `PASS`, verify the Tester Pass Packet matches the current fingerprint,
   then start a separate agent using `$analytics-deployer` and supply both the
   developer and tester packets.
7. Receive the Deployer Completion Packet and verify the PR title, URL, base,
   head, state, updated description, validation section, and remaining checks.
   Confirm sourced motivation, acceptance decisions, and rendered Before/After
   DAG images when required before reporting the PR ready for review.
   Do not require a separate validation comment or comment URL.
8. Report the completed authorized actions and remaining limitations. Ask about
   exact schema cleanup only when no retention decision is already on record.

After integrating substantial child-PR or base-branch changes, reassess the
combined diff, dependencies, deployment path, and affected evidence. Separate
passing checks do not establish an integrated pass. Before handoff, use the
[review checklist](references/review-checklist.md) to quantify findings and
remove unnecessary code, tests, dependencies, and documentation within scope.

For reporting dependency deprecations, read
[reporting-impact.md](references/reporting-impact.md). Have the developer map
the retiring objects and the tester use relevant reporting APIs to find
impacted saved visualizations, dashboards, and other content. This required
dependency check does not authorize downstream data testing or content changes.
Resolve unknown access/coverage and unaddressed breakage before accepting a
retirement pass; do not waive the check merely because validation is light.

Keep detailed evidence internal. The deployer follows
[pull-request-workflow.md](references/pull-request-workflow.md) to maintain the
original description with current outcomes, reconciled comparisons, acceptance
decisions, and required DAG images. Preserve unrelated reviewer content; do not
post separate validation comments unless requested.

## Preserve role boundaries

- Let only the developer edit implementation files.
- Keep the tester independent and read-only with respect to implementation and
  PR state.
- Do not let the deployer proceed without a tester `PASS` tied to the current
  code fingerprint.
- Start or re-engage each specialist and receive its packet directly.
- Avoid concurrent writes to the same worktree, schema, target directory, or
  log directory.
- If separate agents are unavailable, execute the roles sequentially with
  explicit role transitions and packets, and disclose that independence was
  reduced.

## Respect authority and evidence

Read every applicable repository instruction file before acting. Repository
business definitions, modeling architecture, SQL style, dbt commands, PR
template, and contribution workflow override portable defaults.

Treat commits, pushes, PR-description updates, comments, PR creation, readiness
changes, merges, production runs, task-system updates, and schema deletion as
distinct actions.
Honor authorization already explicit in the user's request; ask before an
action that was not authorized. Never imply that a skipped validation passed.

Keep production read-only during pre-PR validation. A role named deployer does
not authorize production deployment. After PR creation, retain isolated schemas
under the user's recorded retention decision; never repeat an already answered
cleanup question or delete schemas without authorization.
