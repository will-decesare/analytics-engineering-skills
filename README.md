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
