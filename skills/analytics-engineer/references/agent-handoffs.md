# Analytics Agent Handoffs

Use structured packets so each role acts on the same ticket and code state.
Communicate directly between agents when collaboration tools are available; do
not make the user relay routine status between roles.

## Shared identity

Every packet must include:

- workflow or ticket identifier and link when available;
- iteration number;
- repository, branch, worktree, and intended base branch;
- current commit SHA plus a fingerprint of staged and unstaged changes;
- isolated schema names and dbt artifact/log paths;
- authority already granted for commits, pushes, PRs, comments, and cleanup;
- packet author role and timestamp.

Do not reuse a pass packet after the fingerprint changes.

## Developer Handoff Packet

Include:

- ticket acceptance criteria and exclusions;
- changed SQL, YAML, tests, docs, semantic resources, and BI contracts;
- purpose, grain, stable key, materialization, and refresh behavior per model;
- source ownership, join cardinalities, cutoffs, and null behavior;
- tests added or changed and why they protect the contract;
- expected row, column, measure, history, and downstream changes;
- descendants believed to be affected;
- developer commands and outcomes;
- known risks, assumptions, unavailable checks, and post-merge work.

Address it to the tester with the explicit state `READY_FOR_TEST`.

## Tester Failure Packet

Include:

- status `FAIL` and tested fingerprint;
- failing command, model, relation, grain key, column, or invariant;
- comparison scope and isolated development relation;
- expected and actual results;
- row and per-column difference counts plus safe representative keys;
- downstream effect and severity;
- relevant log or artifact path;
- smallest actionable correction, without editing the developer's code.

Address it to the developer. The developer must acknowledge it, increment the
iteration after a fix, issue a new fingerprint, and return a new Developer
Handoff Packet.

## Tester Pass Packet

Include:

- status `PASS` and tested fingerprint;
- exact parse, compile, lint, clone/defer, build, and test commands and outcomes;
- production baseline timestamp and identical comparison scope;
- changed models and all validated materialized descendants;
- grain keys and development-only, production-only, changed, and matching rows;
- schema changes by model: added, removed, reordered, and retyped columns;
- per-column value comparison results and type-specific profile deltas;
- expected differences with ticket-based explanations;
- new-model checks where no production relation exists;
- downstream, semantic, metric, exposure, and BI validation;
- limitations, cleanup candidates, and post-merge production checks.

Address it to the deployer and attach the latest Developer Handoff Packet.

## Deployer Completion Packet

Include:

- current code fingerprint and matching tester pass fingerprint;
- commit and push actions actually performed;
- PR title, URL, base, head, and draft or ready state;
- PR body source and validation comment URL when posted;
- CI or dbt job state, never implying pending checks passed;
- remaining pre-merge and post-merge work;
- exact isolated schemas still present;
- whether the user has been asked about schema cleanup.

Tell the user that the PR is ready for review only when the PR actually exists,
the body has been verified, and all actions required by the authorized scope are
complete.

## Coordination rules

- Keep the developer as the only role that edits implementation code.
- Keep the tester read-only with respect to the implementation and PR state.
- Start the deployer only after a current `PASS` packet exists.
- On tester failure, return to development and repeat until pass or a genuine
  blocker requires user input.
- Avoid simultaneous writes to one worktree or dbt artifact directory.
- Report unavailable evidence and stale state; never fill packet fields with
  assumptions.
