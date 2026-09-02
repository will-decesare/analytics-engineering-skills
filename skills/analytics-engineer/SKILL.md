---
name: analytics-engineer
description: Coordinate a role-separated analytics engineering workflow across an analytics developer, independent tester, and pull-request deployer. Use for end-to-end SQL or dbt work tickets that must move from implementation and tests through full development-versus-production and downstream column validation, repair loops, and an evidence-backed draft pull request, or when routing work to one of the specialized analytics roles.
---

# Analytics Engineering Workflow

Coordinate three independent roles while preserving repository rules, test
integrity, and explicit authority for external actions.

## Route to the correct role

- Use `$analytics-developer` to start a work ticket, implement or refactor
  models, add tests, and own correction iterations.
- Use `$analytics-tester` to validate an existing developer handoff against
  production, including changed columns and affected downstream models.
- Use `$analytics-deployer` only after a current tester pass to prepare the PR
  body, validation comment, and authorized draft pull request.

The preferred entry point for a new ticket is `$analytics-developer`. When this
coordinator is invoked for the complete workflow, start a separate developer
agent instructed to use `$analytics-developer` and supply the ticket and
repository context.

## Run the workflow

1. The developer reads the ticket and repository instructions, then implements
   or refactors the required models, metadata, documentation, and tests.
2. The developer completes parse, compile, and lint checks, records the exact
   code fingerprint, and sends a Developer Handoff Packet to a separate tester.
3. The tester uses `$analytics-tester` to build the isolated development state
   and perform full development-versus-production validation on changed models
   and all materially affected downstream models.
4. The tester measures both schema-level column changes and per-column value
   changes, in addition to row-grain comparisons, dbt tests, and business
   invariants.
5. On `FAIL`, the tester sends an actionable failure packet to the developer.
   The developer fixes the code, increments the iteration, and returns it for a
   complete retest. Repeat until `PASS` or a genuine blocker needs user input.
6. On `PASS`, the tester sends the current Developer Handoff Packet and Tester
   Pass Packet to a separate deployer using `$analytics-deployer`.
7. The deployer verifies that the tested fingerprint is current, populates the
   repository PR template and validation comment, performs only authorized Git
   and GitHub actions, and creates a draft PR when authorized.
8. After verifying the created PR, the deployer tells the user that it is ready
   for review and asks whether the exact isolated development schemas should be
   dropped.

Read [agent-handoffs.md](references/agent-handoffs.md) for the required packets
and communication protocol.

## Preserve role boundaries

- Let only the developer edit implementation files.
- Keep the tester independent and read-only with respect to implementation and
  PR state.
- Do not let the deployer proceed without a tester `PASS` tied to the current
  code fingerprint.
- Communicate directly between agents when collaboration tools are available;
  do not require the user to relay normal handoffs.
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
