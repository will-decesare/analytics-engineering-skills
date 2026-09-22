# Analytics Engineering Skills

A coordinated suite of reusable Codex skills for analytics engineering and dbt
workflows.

## Skills

- `analytics-engineer`: serves as the quarterback and default entry point. It
  owns the ticket from intake through implementation, validation loops, and the
  final pull-request handoff.
- `analytics-developer`: implements work tickets, refactors or creates models,
  and adds relevant tests and documentation.
- `analytics-tester`: independently compares development with production at the
  row, schema, and column-value levels across changed and downstream models.
- `analytics-deployer`: prepares evidence-backed pull requests and validation
  comments after testing passes.

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
pull request. You may commit, push, create the draft pull request, and post its
validation comment after the tester passes.
```

The coordinator starts each specialist and keeps the current ticket, code
fingerprint, validation state, and authorization scope. Failed validation is
routed back to the developer. After a passing tester handoff, the coordinator
starts the deployer, verifies the resulting pull request, reports that it is
ready for review, and asks whether isolated development schemas should be
dropped.

## Model routing

Start the coordinator on the strong default in the shared routing policy.
The workflow also uses strong models for development and independent testing,
with cheaper models for bounded routine evidence collection and Git/PR
preparation after a verified tester pass.
The tester still designs checks, interprets evidence, and owns the verdict.
Ambiguous evidence returns to a strong specialist; complex Git/PR work can
escalate to a stronger deployer.

These defaults are applied at agent launch, not through skill UI metadata.
Explicit user model choices take precedence. Direct skill invocation retains
the caller's model; unavailable model selection falls back with disclosure.
Batch evidence work only when delegation is useful, since extra agents and
retries can offset savings. See the shared
[routing policy](skills/analytics-engineer/references/model-routing.md) for
launch settings, role boundaries, and escalation rules.
