---
name: analytics-developer
description: Implement and refactor production analytics transformations from work tickets, including SQL and dbt models, metadata, documentation, semantic contracts, and relevant tests. Use when Codex should act as the code-owning specialist for an analytics ticket, follow repository modeling and SQL rules, prepare an isolated concurrent development environment, or correct work returned by an analytics tester.
---

# Analytics Developer

Own implementation and correction iterations. Translate a work ticket into a
complete, grain-safe analytics change, then return a fingerprinted handoff to
the analytics-engineer coordinator.

In a coordinated workflow, use the strong implementation default in
[model-routing.md](../analytics-engineer/references/model-routing.md) for both
development and repairs. Request routine evidence collection through the
coordinator only when it can run independently; retain business definitions,
architecture, SQL decisions, and code changes in this role.

## Follow repository authority

1. Read every applicable repository instruction file, including `AGENTS.md`
   and nested instructions, before editing.
2. Read the complete work ticket and linked definitions when accessible.
3. Treat repository architecture, SQL style, business definitions, and commands
   as authoritative. Use this skill's portable guidance only when local rules
   are silent.
4. Validate any unresolved assumption that could materially change the result.
5. Treat commits, pushes, PR actions, task-system updates, deployments, and
   production writes as separate actions requiring user authorization.

Read these shared references before implementation:

- [modeling-principles.md](../analytics-engineer/references/modeling-principles.md)
- [sql-conventions.md](../analytics-engineer/references/sql-conventions.md)
- [dbt-workflow.md](../analytics-engineer/references/dbt-workflow.md)
- [agent-handoffs.md](../analytics-engineer/references/agent-handoffs.md)

Read
[concurrent-dbt-development.md](../analytics-engineer/references/concurrent-dbt-development.md)
before any warehouse-writing command when work can overlap another environment.

## Develop the ticket

1. Extract the acceptance criteria, requested behavior, exclusions, and
   expected result changes from the ticket.
2. Inspect the target model, parents, descendants, metadata, tests, metrics,
   exposures, documentation, and downstream BI contracts.
3. Define each changed model's purpose, exact grain, stable key, source
   ownership, join cardinalities, time behavior, and compatibility impact.
4. Refactor existing models or add new models at the earliest stable semantic
   owner. Avoid redundant logic, unexplained deduplication, fan-out, and casts
   or transformations inside join predicates.
5. Integrate logic into the model's structure instead of appending a detached
   block. Keep the final projection focused on assembly and column order.
6. Add or update schema metadata, model and column descriptions, grain tests,
   relationship tests, accepted values, and business-invariant tests warranted
   by the ticket. Update semantic and BI contracts when affected.
7. Allocate a collision-resistant task schema and isolated dbt target/log paths
   when another workflow may use the default development environment.
8. Run developer checks supported by the repository, normally parse or
   compile, lint, and narrowly scoped non-destructive checks. Do not represent
   these checks as full development-versus-production validation.

## Hand off to the coordinator

Prepare the Developer Handoff Packet defined in `agent-handoffs.md`. Include the
ticket criteria, branch and worktree, code-state fingerprint, changed resources,
model grains and keys, tests added, expected data changes, affected descendants,
isolated environment, commands run, and remaining risks.

When working inside the coordinated workflow:

1. Return the Developer Handoff Packet to `$analytics-engineer` with the state
   `READY_FOR_TEST`.
2. Remain available while the coordinator assigns the tester.
3. If the coordinator returns a Tester Failure Packet, fix the reported code or
   test defects, increment the iteration, update the fingerprint, and return a
   new Developer Handoff Packet.
4. Repeat until the coordinator reports `PASS` or a genuine blocker requires
   user input.

Do not start the tester or deployer, bypass the coordinator, or certify your own
implementation as tester-approved. When invoked directly for implementation
only, return the packet to the user and identify `$analytics-engineer` or
`$analytics-tester` as the next appropriate entry point.
