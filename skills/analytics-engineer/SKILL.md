---
name: analytics-engineer
description: Coordinate analytics work tickets through development, changed-column validation, repair loops, and a concise evidence-backed PR. Use as the default analytics workflow entry point. Downstream data validation is opt-in; reporting deprecations require live content-impact checks.
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
  production, testing only changed columns in directly changed models by
  default. Include downstream data validation only when explicitly requested;
  require live reporting-impact checks for deprecations as described below.
- Use `$analytics-deployer` only after a current tester pass to prepare the PR
  description with its validation summary and the authorized draft pull
  request, or update that same description when the PR already exists.

As quarterback, retain ownership of the ticket, current code fingerprint,
validation status, authorization scope, and next role. Start each specialist,
receive every handoff, and route the next action without requiring the user to
coordinate agents.

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

1. Read the work ticket and repository instructions, establish the workflow
   identity and authorization scope, then start a separate agent using
   `$analytics-developer`. Require the developer to complete the
   [branch preflight](references/dbt-workflow.md#start-from-current-remote-main)
   before creating a branch or worktree from a production base: fetch remote
   `main`, compare the local base, and start from the verified current tip.
2. Receive the Developer Handoff Packet, verify its code fingerprint and
   `READY_FOR_TEST` state, and record the model/changed-column allowlist before
   starting a separate `$analytics-tester` agent.
3. Have the tester build the isolated development state and perform full
   development-versus-production validation only for that allowlist. Do not
   validate downstream data unless the user explicitly requests it. For
   reporting deprecations, also require the live content-impact check below.
4. Require schema and value comparisons and relevant tests for changed columns
   only. Use stable keys for row alignment and minimal comparison-integrity
   checks; exclude unchanged-column profiling and unrelated test suites.
   For uncertain acceptance or logic-driven metric changes not explicitly
   accepted by the user, ask with the observed impact and proposed explanation.
   Continue independent work, but keep the affected validation gate unresolved.
5. On `FAIL`, receive the Tester Failure Packet and route it to the developer.
   After the fix, require a new fingerprint and retest the updated allowlist.
   Repeat until `PASS` or a genuine blocker needs user input.
6. On `PASS`, verify the Tester Pass Packet matches the current fingerprint,
   then start a separate agent using `$analytics-deployer` and supply both the
   developer and tester packets.
7. Receive the Deployer Completion Packet and verify the PR title, URL, base,
   head, state, updated description, validation section, and remaining checks.
   Confirm sourced motivation, acceptance decisions, and rendered Before/After
   DAG images when required before reporting the PR ready for review.
   Do not require a separate validation comment or comment URL.
8. Tell the user that the PR is ready for review and ask whether the exact
   isolated development schemas should be dropped.

Read [agent-handoffs.md](references/agent-handoffs.md) for the required packets
and communication protocol.

Read [full-validation.md](references/full-validation.md) to resolve scope.
"Full validation" means complete validation within the changed-column
allowlist, not all columns or descendants. Record unrequested downstream work
as excluded by scope, not incomplete validation or a reason to block a PR.

For reporting dependency deprecations, read
[reporting-impact.md](references/reporting-impact.md). Have the developer map
the retiring objects and the tester use relevant reporting APIs to find
impacted saved visualizations, dashboards, and other content. This required
dependency check does not authorize downstream data testing or content changes.
Resolve unknown access/coverage and unaddressed breakage before accepting a
retirement pass; do not waive the check merely because validation is light.

Keep fingerprint verification and detailed evidence in internal handoffs.
Require the deployer to maintain one concise validation summary in the original
PR description, preserving unrelated content. New results, retests, and
reporting-impact findings update that section in place; do not post separate
"Validation update" comments unless the user explicitly requests a comment.
Lead with parse/compile/run/test outcomes, then changed-column comparison
tables and material metric differences with reasons and acceptance decisions.
Only report downstream execution when explicitly requested and actually run.
For DAG changes, require real before/after dbt Docs screenshots under
[dag-screenshots.md](references/dag-screenshots.md).
Exclude command dumps, JSON artifacts, hashes, and code-provenance narration
from reviewer-facing text unless explicitly requested.

## Preserve role boundaries

- Let only the developer edit implementation files.
- Keep the tester independent and read-only with respect to implementation and
  PR state.
- Do not let the deployer proceed without a tester `PASS` tied to the current
  code fingerprint.
- Require every specialist to return its packet to the coordinator. The
  coordinator starts or re-engages the next role and maintains workflow state.
- Communicate directly with specialists when collaboration tools are
  available; do not require the user to relay normal handoffs.
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
until the user answers the explicit cleanup question.
