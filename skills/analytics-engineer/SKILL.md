---
name: analytics-engineer
description: Serve as the quarterback for a role-separated analytics engineering workflow across an analytics developer, independent tester, and pull-request deployer. Use as the default entry point for end-to-end SQL or dbt work tickets that must move from implementation and tests through full development-versus-production and downstream column validation, repair loops, and an evidence-backed draft pull request.
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
  production, including changed columns and affected downstream models.
- Use `$analytics-deployer` only after a current tester pass to prepare the PR
  body, validation comment, and authorized draft pull request.

As quarterback, retain ownership of the ticket, current code fingerprint,
validation status, authorization scope, and next role. Start each specialist,
receive every handoff, and route the next action without requiring the user to
coordinate agents.

## Run the workflow

1. Read the work ticket and repository instructions, establish the workflow
   identity and authorization scope, then start a separate agent using
   `$analytics-developer`.
2. Receive the Developer Handoff Packet, verify its code fingerprint and
   `READY_FOR_TEST` state, then start a separate `$analytics-tester` agent.
3. Have the tester build the isolated development state and perform full
   development-versus-production validation on changed models and all
   materially affected downstream models.
4. Require both schema-level column comparisons and per-column value
   comparisons, in addition to row-grain checks, dbt tests, and business
   invariants.
5. On `FAIL`, receive the Tester Failure Packet and route it to the developer.
   After the fix, require a new fingerprint and complete retest. Repeat until
   `PASS` or a genuine blocker needs user input.
6. On `PASS`, verify the Tester Pass Packet matches the current fingerprint,
   then start a separate agent using `$analytics-deployer` and supply both the
   developer and tester packets.
7. Receive the Deployer Completion Packet and verify the PR title, URL, base,
   head, state, body, validation comment, and remaining checks.
8. Tell the user that the PR is ready for review and ask whether the exact
   isolated development schemas should be dropped.

Read [agent-handoffs.md](references/agent-handoffs.md) for the required packets
and communication protocol.

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

Treat commits, pushes, comments, PR creation, readiness changes, merges,
production runs, task-system updates, and schema deletion as distinct actions.
Honor authorization already explicit in the user's request; ask before an
action that was not authorized. Never imply that a skipped validation passed.

Keep production read-only during pre-PR validation. A role named deployer does
not authorize production deployment. After PR creation, retain isolated schemas
until the user answers the explicit cleanup question.
