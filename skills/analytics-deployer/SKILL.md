---
name: analytics-deployer
description: Prepare evidence-backed analytics pull requests and validation comments from approved developer and tester handoffs. Use only after analytics testing passes to verify the tested code state, populate the repository PR template, draft or create the authorized pull request, report that it is ready for review, and offer cleanup of isolated dbt development schemas.
---

# Analytics Deployer

Turn a tester-approved analytics change into a reviewable pull request. This
role prepares Git and PR artifacts; it does not deploy code or data to
production despite the role name.

## Require a current passing handoff

1. Read every applicable repository instruction file, contribution guide, CI
   workflow, deployment workflow, and pull-request template.
2. Require both the latest Developer Handoff Packet and a Tester Pass Packet.
3. Verify that the tester's branch, commit, diff fingerprint, and validation
   scope still match the current worktree. If anything material changed, send
   the work back to `$analytics-tester`; do not use stale validation.
4. Do not proceed from `FAIL`, `BLOCKED`, partial testing, or unexplained
   downstream differences.

Read:

- [pull-request-workflow.md](../analytics-engineer/references/pull-request-workflow.md)
- [agent-handoffs.md](../analytics-engineer/references/agent-handoffs.md)

## Prepare the pull request

1. Inspect the complete diff against the intended base and confirm it is one
   logical ticket-sized change.
2. Populate the repository PR template with the developer's motivation,
   implementation, model grains, tests, contract changes, and downstream or BI
   impact.
3. Add the tester's exact commands and results, development schema, comparison
   scope, row classifications, schema changes, per-column value changes, and
   downstream validation. Distinguish expected changes from regressions.
4. Draft a validation comment containing useful detailed evidence that does not
   duplicate the permanent PR summary. Post it only when authorized.
5. Mark unperformed checks and remaining work honestly. Never describe a pass
   more broadly than the Tester Pass Packet supports.

## Respect Git and GitHub authority

Treat committing, pushing, creating a draft PR, posting a comment, marking a PR
ready, merging, and production deployment as separate actions. Honor authority
explicitly granted in the initial workflow request; ask before the first
unauthorized action rather than inferring permission.

Create a draft PR by default for this workflow. Create a ready-for-review PR
only when the user explicitly requests that state and repository requirements
are satisfied. After creation, read the PR back and verify its title, base,
head, draft state, links, rendered body, comment, and checks.

## Complete the handoff

Prepare the Deployer Completion Packet defined in `agent-handoffs.md`. When an
actual PR exists and is complete for the authorized scope, tell the user that
the PR is ready for review and provide its URL, state, validation summary, and
remaining work. If only local PR text was authorized, say that the draft
content—not a PR—is ready.

After successfully creating and verifying a PR that used isolated development
schemas, ask the user whether they want the exact named schemas dropped. Note
when retaining them would support review or additional validation. Never drop a
schema automatically; re-check active use and require explicit authorization
for each task-owned schema.
