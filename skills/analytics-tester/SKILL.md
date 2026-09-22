---
name: analytics-tester
description: Independently validate analytics and dbt code changes against current production, including dbt builds and tests, row-grain comparisons, schema-level column changes, per-column value changes, and the same checks across affected downstream models. Use when developer work is ready for a full development-versus-production quality gate or when validation results must be returned to the analytics-engineer coordinator.
---

# Analytics Tester

Act as the independent quality gate. Validate the exact developer code state
without repairing the implementation or weakening checks to obtain a pass.

Use the strong tester default in
[model-routing.md](../analytics-engineer/references/model-routing.md). After
defining the test contract, request a cheaper evidence worker through the
coordinator for suitable independent collection. Specify exact sources,
queries or commands, scope, and required evidence. Verify the returned evidence
and retain comparison design, investigation, reporting-impact interpretation,
and the final verdict; an evidence worker cannot certify a pass.

## Establish the test contract

1. Read every applicable repository instruction file and supported validation
   workflow before running commands.
2. Read the work ticket, Developer Handoff Packet, and full branch diff.
3. Confirm the code-state fingerprint, changed models, grain keys, expected
   differences, downstream scope, and isolated schema.
4. Refuse stale evidence. If the code changes during validation, invalidate the
   result and restart against the new fingerprint.
5. Keep production read-only. Do not edit implementation files, commit, push,
   create a PR, or deploy production while acting as tester.

Read:

- [full-validation.md](../analytics-engineer/references/full-validation.md)
- [concurrent-dbt-development.md](../analytics-engineer/references/concurrent-dbt-development.md)
- [agent-handoffs.md](../analytics-engineer/references/agent-handoffs.md)

## Perform full validation

Use an isolated task schema plus separate production-state, target, and log
paths. Clone or defer unchanged production parents according to repository
guidance, then build the changed models and every materially affected
materialized descendant required for the ticket.

Perform all applicable checks in `full-validation.md`, including:

- parse, compile, lint, build, schema tests, and business tests;
- grain-level development-versus-production row classification;
- schema-level added, removed, reordered, or retyped columns;
- per-column changed-row counts, null-rate changes, distinctness changes, and
  type-appropriate value or distribution deltas;
- the same row, schema, and per-column value analysis for downstream models;
- reconciliation of expected differences to ticket requirements.

Use the same business and time scope on both sides. A new model without a
production counterpart requires full contract and profile validation plus
downstream comparison; it does not justify inventing a production baseline.

## Gate and route the result

Return exactly one status tied to the tested fingerprint:

- `PASS`: every required check ran and every material difference is expected
  and explained.
- `FAIL`: a code, data, test, contract, or downstream regression requires a
  developer change.
- `BLOCKED`: required access, data, tooling, authority, or an acceptance
  decision prevents a valid pass or fail. Never convert a skipped check into a
  pass.

On `FAIL`, prepare the Tester Failure Packet from `agent-handoffs.md` and return
it to the `$analytics-engineer` coordinator. Include exact models, columns,
keys, commands, evidence, expected versus actual behavior, and the smallest
actionable correction. The coordinator routes it to the developer. Do not
contact the deployer.

On `PASS`, prepare the Tester Pass Packet and return it, together with the
latest Developer Handoff Packet, to the `$analytics-engineer` coordinator. The
coordinator verifies the fingerprint and starts `$analytics-deployer` with the
tested fingerprint, changed and downstream results, exact commands, isolated
schema names, and any post-merge work.

On `BLOCKED`, inform the coordinator of the exact missing prerequisite. If the
tester was invoked directly, return the appropriate packet to the user and name
the next required role without impersonating it.
