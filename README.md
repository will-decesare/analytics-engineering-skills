# Analytics Engineering Skills

A coordinated suite of reusable Codex skills for analytics engineering and dbt
workflows.

## Skills

- `analytics-engineer`: serves as the quarterback and default entry point. It
  owns the ticket from intake through implementation, validation loops, and the
  final pull-request handoff.
- `analytics-developer`: implements work tickets, refactors or creates models,
  and adds relevant tests and documentation.
- `analytics-tester`: independently compares changed columns in directly
  changed models with production, including relevant tests, schema checks, and
  row-keyed value comparisons. Downstream data validation is opt-in; reporting
  deprecations require a live API-based content-impact check.
- `analytics-deployer`: prepares evidence-backed pull requests and maintains
  concise validation results in the original PR description after testing.

Install all four skills together because the specialized roles use shared
references from `analytics-engineer`.

```bash
mkdir -p ~/.codex/skills
cp -R skills/analytics-* ~/.codex/skills/
```

Start a work ticket with:

```text
Use $analytics-engineer to tackle <ticket link or requirements> through
development, validation, and a draft pull request.
```

To authorize the complete Git and pull-request flow upfront:

```text
Use $analytics-engineer to handle <ticket> through full testing and a draft
pull request. You may commit, push, create the draft pull request, and update
its description with validation results after the tester passes.
```

The coordinator starts each specialist and keeps the current ticket, code
fingerprint, validation state, and authorization scope. Failed validation is
routed back to the developer. After a passing tester handoff, the coordinator
starts the deployer, verifies the resulting pull request, reports that it is
ready for review, and asks whether isolated development schemas should be
dropped.

## Model routing

Start the coordinator on the latest, most capable available model for complex
agent work, following the Strong tier in the shared routing policy.
The workflow also uses strong models for development and independent testing,
with current smaller, faster, lower-cost models for bounded evidence collection
and Git/PR preparation after a verified tester pass.
The tester still designs checks, interprets evidence, and owns the verdict.
Ambiguous evidence returns to a strong specialist; complex Git/PR work can
escalate to a stronger deployer.

These defaults are applied at agent launch, not through skill UI metadata.
Resolve the tiers from the runtime's available models for each new workflow,
so new model releases do not require editing pinned names in the skills.
Explicit user model choices take precedence. Direct skill invocation retains
the caller's model; unavailable model selection falls back with disclosure.
Batch evidence work only when delegation is useful, since extra agents and
retries can offset savings. See the shared
[routing policy](skills/analytics-engineer/references/model-routing.md) for
launch settings, role boundaries, and escalation rules.

## Validation scope

Full validation covers only changed columns in directly changed models by
default. Unchanged columns are excluded from profiling and value comparisons;
row keys still align the comparison. All columns of a new model count as added
columns. Downstream models and columns are not built or validated unless you
explicitly request it, and their exclusion does not block a passing result or
PR preparation.

When retiring a reporting dependency, the developer maps its reporting
identifiers and the tester uses the relevant reporting APIs to check saved
visualizations, dashboards, and related content. This does not enable broad
downstream data testing. Missing live access is reported as unknown impact,
not proof of safety. Hosted content is not changed without authorization.

Validation results, material differences, reporting impact, and remaining
actions belong in the original PR description. Subsequent results update its
existing sections while preserving unrelated content; no separate validation
comments are posted unless explicitly requested. Detailed commands, JSON,
local artifact paths, and commit/fingerprint verification stay in internal
handoffs.

PR descriptions explain both the change and its motivation; missing motivation
is requested from the user. Validation starts with separate dbt parse, compile,
run, and test summaries for the actual scope, followed by changed-column and
material metric comparison tables with reasons. Rounding and freshness
differences need evidence; logic-driven metric changes need explicit user
acceptance. Downstream checks, including immediate downstream execution, remain
opt-in. Negligible accepted differences can be summarized without numeric deltas.

For dependency changes, generate and serve before/after dbt Docs sites, capture
their actual DAGs, and embed verified images in Before/After dropdowns. PR to-do
lists focus on release coordination and exceptional post-merge actions; routine
smoke testing stays out of the description.
