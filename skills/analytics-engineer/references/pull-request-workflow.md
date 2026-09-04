# Pull Request Workflow

## Contents

- [Establish authority and repository rules](#establish-authority-and-repository-rules)
- [Choose the PR body or a comment](#choose-the-pr-body-or-a-comment)
- [Gather evidence](#gather-evidence)
- [Populate the PR description](#populate-the-pr-description)
- [Populate a validation comment](#populate-a-validation-comment)
- [Draft or create the PR](#draft-or-create-the-pr)
- [Verify and maintain the PR](#verify-and-maintain-the-pr)

## Establish authority and repository rules

- Treat committing, pushing, commenting, creating a PR, marking it ready, and
  merging as separate external actions. Perform only the actions the user
  authorized.
- Read repository instruction files, the PR template, contribution guidance,
  CI workflows, deployment workflow, and task-management conventions.
- Use the configured base branch. If none is documented, default to the
  repository's default branch.
- Follow local branch naming. If no convention exists, use a concise
  `feat/`, `fix/`, or `refactor/` name that matches the change type.
- Link the issue, task, specification, or prior PR when one exists. Preserve its
  identifier in the title, body, or commits only where local rules require it.

## Choose the PR body or a comment

Put durable context in the PR body:

- motivation and scope;
- model grain and contract changes;
- implementation approach;
- development-versus-production validation;
- migration, full-refresh, removal, or downstream BI impact;
- work required before or after merge.

Use a PR comment for incremental evidence or discussion:

- validation completed after the PR was opened;
- reviewer-requested analysis;
- updated row-level comparison results;
- deployment or smoke-test status;
- a material correction whose history should remain visible.

Edit the PR body instead of adding a comment when permanent summary information
was missing or became stale. Avoid posting the same content in both places.

## Gather evidence

Before drafting either artifact:

1. inspect the full branch diff against the intended base branch;
2. verify that the branch represents one logical piece of work;
3. list changed models, metadata, tests, macros, docs, and downstream contracts;
4. identify the grain and stable key of every changed output;
5. collect exact parse, compile, lint, build, and test commands and outcomes;
6. collect row-level development-versus-production results, including the
   isolated development schema, scope, relation names, difference classes, and
   representative keys;
7. collect schema-level and per-column value changes for every changed model
   and materially affected downstream model, including added, removed, and
   retyped columns, changed-row counts, null rates, distinctness, and relevant
   type-specific profile deltas;
8. identify expected differences and explain why they are correct;
9. capture DAG or BI screenshots and links when the repository requests them;
10. identify pre-merge and post-merge work, including full refreshes, object
   cleanup, backfills, smoke tests, and stakeholder communication.

Do not paste secrets, credentials, private customer data, or excessive raw logs.
Prefer summarized evidence with links or collapsible details.

## Populate the PR description

Use the repository's template as the authoritative structure. Keep its required
headings, remove only sections explicitly marked optional, and mark checklist
items complete only when they are true. A portable analytics PR body commonly
contains:

```markdown
## Description and Motivation

<What changed, why it changed, and links to tasks or related PRs.>

## To-do before merge

- [ ] <Only unresolved pre-merge work; omit if empty and optional.>

## Screenshots

<Before/after DAG or BI evidence when useful.>

## Validation of models

- Static checks: `<commands and outcomes>`
- Development build/tests: `<commands and outcomes>`
- Development vs. production row comparison:
  `<key, scope, difference counts, and explanation>`
- Column-level changes:
  `<schema and per-column value changes by model>`
- Downstream validation:
  `<descendant row and column comparisons, dashboards, or limitations>`

## Changes to existing models

<Grain, columns, semantic contracts, BI changes, compatibility, and any
full-refresh or removal instructions.>

## To-do after merge

- [ ] <Production validation, refresh, smoke test, cleanup, or communication.>

## Checklist

- [ ] The PR is one logical piece of work.
- [ ] Models are materialized appropriately.
- [ ] New models and columns have appropriate tests and documentation.
- [ ] Unique keys or unique column combinations test the declared grain.
```

Use a descriptive title that summarizes the analytics behavior change. State
whether results, grain, schema, history, or downstream contracts change. Do not
describe an intended result as validated unless the cited check actually ran.

## Populate a validation comment

Use this compact structure and omit irrelevant rows:

```markdown
## Validation update

**Scope:** `<models, selector, commit, and comparison window>`

| Check | Result |
| --- | --- |
| Parse/compile | `<pass/fail/not run + command>` |
| Lint | `<pass/fail/not run + command>` |
| Development build/tests | `<pass/fail/not run + command>` |

### Development vs. production row comparison

| Model | Grain key | Dev rows | Prod rows | Dev only | Prod only | Changed |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| `<model>` | `<key>` | `<n>` | `<n>` | `<n>` | `<n>` | `<n>` |

**Expected differences:** `<what changed and why>`

### Schema and column-value changes

| Model | Column | Contract change | Changed rows | Dev null % | Prod null % | Profile delta |
| --- | --- | --- | ---: | ---: | ---: | --- |
| `<model>` | `<column>` | `<added/removed/retyped/none>` | `<n or n/a>` | `<n>` | `<n>` | `<type-appropriate summary>` |

Include changed models and materially affected downstream models. Summarize
unchanged columns compactly and link to protected full evidence when the table
would be excessive.

<details>
<summary>Representative changed keys</summary>

`<safe sample keys or a link to protected results>`

</details>

**Downstream validation:** `<models, dashboards, or limitations>`

**Remaining work:** `<production checks or none>`
```

Use the same scope on both sides of the row comparison. Explain every material
difference class; do not present matching aggregate counts as row-level parity.
Redact sensitive row contents and link to access-controlled evidence when
samples cannot be posted safely.

## Draft or create the PR

1. Confirm the current branch, intended base branch, clean staging state, and
   complete diff.
2. Confirm commits are present and related. Do not create or rewrite commits
   unless the user requested that work.
3. Push the branch only with authorization.
4. Write the completed PR body to a temporary file to preserve Markdown and
   avoid shell-escaping errors.
5. Create a draft PR when validation, screenshots, review preparation, or
   pre-merge work remains. Create a ready PR only when the repository checklist
   is satisfied and the user requested or clearly authorized a ready PR.
6. With GitHub CLI, prefer:

   ```text
   gh pr create --draft --base <base> --head <branch> \
       --title "<descriptive title>" --body-file <body-file>
   ```

   Remove `--draft` only for an authorized ready PR. Use the repository's
   purpose-built connector instead when it is the established workflow.

## Verify and maintain the PR

- Read the created PR back and verify its title, base, head, draft state, links,
  Markdown rendering, and checklist.
- If the work used an isolated development schema, include the exact task-owned
  schema names and retention considerations in the Deployer Completion Packet.
  After the PR is successfully created and verified, the analytics-engineer
  coordinator asks the user whether to drop them. Do not drop anything without
  explicit authorization, and re-check active use before cleanup.
- Inspect CI or dbt job status; do not claim success while checks are pending.
- Add a validation comment only when it contributes new evidence. Prefer
  updating a previous bot/agent comment over creating repetitive comments when
  the platform supports it.
- Update the body when scope, migration instructions, downstream impact, or
  permanent validation conclusions change.
- After merge, follow the documented deployment workflow and complete the
  production row-level validation and to-do list. Do not deploy or mutate
  production merely because the PR merged.
