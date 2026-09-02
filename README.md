# Analytics Engineering Skills

A coordinated suite of reusable Codex skills for analytics engineering and dbt
workflows.

## Skills

- `analytics-developer`: implements work tickets, refactors or creates models,
  and adds relevant tests and documentation.
- `analytics-tester`: independently compares development with production at the
  row, schema, and column-value levels across changed and downstream models.
- `analytics-deployer`: prepares evidence-backed pull requests and validation
  comments after testing passes.
- `analytics-engineer`: coordinates the complete developer-to-tester-to-deployer
  workflow.

Install all four skills together because the specialized roles use shared
references from `analytics-engineer`.

```bash
mkdir -p ~/.codex/skills
cp -R skills/analytics-* ~/.codex/skills/
```

Start a work ticket with:

```text
Use $analytics-developer to tackle <ticket link or requirements>.
```

To authorize the complete Git and pull-request flow upfront:

```text
Use $analytics-developer to handle <ticket> through full testing and a draft
pull request. You may commit, push, create the draft pull request, and post its
validation comment after the tester passes.
```

The workflow loops failed validation back to the developer. A passing tester
handoff starts the deployer, which creates the authorized pull request, reports
that it is ready for review, and asks whether isolated development schemas
should be dropped.
