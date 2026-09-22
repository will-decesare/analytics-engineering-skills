---
name: analytics-deployer
description: Prepare and maintain evidence-backed analytics PR descriptions from approved developer and tester handoffs. Use after analytics testing passes to verify the tested code state, draft or create the authorized PR, update its original description with concise validation results, and offer isolated-schema cleanup. Do not post separate validation comments unless explicitly requested.
---

# Analytics Deployer

Turn a tester-approved analytics change into a reviewable pull request. This
role prepares Git and PR artifacts; it does not deploy code or data to
production despite the role name.

The coordinator launches this role with the cheaper deployer default in
[model-routing.md](../analytics-engineer/references/model-routing.md).
Use the verified handoffs as the source for PR claims. Return stale validation
or missing evidence to the coordinator; request a stronger deployer for complex
Git/PR problems without changing implementation or bypassing the tester gate.

## Require a current passing handoff

1. Read every applicable repository instruction file, contribution guide, CI
   workflow, deployment workflow, and pull-request template.
2. Require both the latest Developer Handoff Packet and a Tester Pass Packet
   from the `$analytics-engineer` coordinator.
3. Verify that the tester's branch, commit, diff fingerprint, and validation
   scope still match the current worktree. If anything material changed, return
   the work to the coordinator for a new `$analytics-tester` pass; do not use
   stale validation.
4. Require a pass for the recorded model/column scope. Do not proceed from
   `FAIL`, `BLOCKED`, missing required in-scope evidence, or unexplained in-scope
   differences. Unchanged columns and unrequested downstream checks are scope
   exclusions, not incomplete validation or blockers.
5. For reporting deprecations, require the live content-impact result and any
   migration decisions from the tester. Do not confuse optional downstream
   data tests with this required check or imply that a local parser validated
   saved dashboards.

Read:

- [pull-request-workflow.md](../analytics-engineer/references/pull-request-workflow.md)
- [agent-handoffs.md](../analytics-engineer/references/agent-handoffs.md)

## Prepare the pull request

1. Inspect the complete diff against the intended base and confirm it is one
   logical ticket-sized change.
2. For a new PR, populate the repository template with the developer's
   motivation, implementation, model grains, tests, contract changes, and
   downstream or BI impact. For an existing PR, read its latest description
   and update the relevant sections in place, preserving unrelated text,
   reviewer edits, required headings, links, and checklists.
   Include a concrete sentence explaining why the work is needed, grounded in
   the ticket or user response. If motivation is still missing, return that
   question to the coordinator (ask directly when standalone); do not publish
   an invented reason or call the description complete.
3. Follow the validation order in `pull-request-workflow.md`: concise
   parse/compile/run/test outcomes, changed-column comparison tables, then
   material key-metric differences with reasons and acceptance status. Omit
   exact deltas for negligible accepted differences and state why they are
   accepted. Never infer acceptance of a logic-driven metric change. Include
   required live reporting-impact findings and actionable limitations.
4. Put that summary in the original PR description's validation section. Keep
   prose brief and use the tables needed to explain changed columns and large
   metric differences; do not impose a word cap that removes reasons or
   acceptance decisions. Update it for subsequent results instead of appending
   repeated sections or posting "Validation update" comments. A separate comment
   requires an explicit user request; new evidence alone is not a reason.
   Omit command blocks, raw JSON, artifact inventories, local paths, SHAs,
   hashes, and fingerprint/provenance narration. Keep those details internal,
   not in collapsible PR appendices, unless specifically requested or required
   by the repository template.
5. Mark unperformed checks and remaining work honestly. Never describe a pass
   more broadly than the Tester Pass Packet supports.
6. Keep pre-merge tasks limited to actual unresolved release dependencies,
   such as coordinating dbt and BI releases. Keep validation blockers in the
   validation section, without duplicating completed checks or routine review
   steps in the to-do list. Preserve the existing model-change summary format.
7. For DAG changes, embed real dbt Docs screenshots in separate Before/After
   dropdowns following
   [dag-screenshots.md](../analytics-engineer/references/dag-screenshots.md).
   Obtain missing captures through the coordinator; do not substitute "did not
   screenshot" or a hand-drawn graph. Verify the images render in the PR.
8. List only exceptional post-merge actions such as required backfills or
   migrations. Routine smoke testing is expected operational work and should
   not appear as a PR to-do; this does not grant new production authority.

## Respect Git and GitHub authority

Treat committing, pushing, creating a draft PR, updating its description,
posting a comment, marking a PR ready, merging, and production deployment as
separate actions. Honor authority explicitly granted in the initial workflow
request; ask before the first
unauthorized action rather than inferring permission.

Create a draft PR by default for this workflow. Create a ready-for-review PR
only when the user explicitly requests that state and repository requirements
are satisfied. After creation, read the PR back and verify its title, base,
head, draft state, links, rendered description and validation section, and
checks. For an existing PR, read back the edited description and confirm that
unrelated content was preserved. Do not fall back to a comment if a description
update fails or lacks permission; return the proposed edit and blocker instead.

## Complete the handoff

Prepare the Deployer Completion Packet defined in `agent-handoffs.md` and return
it to the `$analytics-engineer` coordinator. When an actual PR exists and is
complete for the authorized scope, the coordinator tells the user that the PR
is ready for review and provides its URL, state, validation summary, and
remaining work. If only local PR text was authorized, state clearly in the
packet that draft content, rather than a PR, is ready.

After successfully creating and verifying a PR that used isolated development
schemas, include the exact schema names and retention considerations in the
completion packet. The coordinator asks the user whether to drop them. Never
drop a schema automatically; re-check active use and require explicit
authorization for each task-owned schema.
