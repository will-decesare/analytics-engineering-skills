# Pull Request Workflow

## Contents

- [Establish authority and repository rules](#establish-authority-and-repository-rules)
- [Maintain the original PR description](#maintain-the-original-pr-description)
- [Gather evidence](#gather-evidence)
- [Establish the motivation](#establish-the-motivation)
- [Keep merge tasks actionable](#keep-merge-tasks-actionable)
- [Populate the PR description](#populate-the-pr-description)
- [Write the validation section](#write-the-validation-section)
- [Draft or create the PR](#draft-or-create-the-pr)
- [Verify and maintain the PR](#verify-and-maintain-the-pr)

## Establish authority and repository rules

- Treat committing, pushing, updating a PR description, commenting, creating a
  PR, marking it ready, and merging as separate external actions. Perform only
  the actions the user authorized.
- Read repository instruction files, the PR template, contribution guidance,
  CI workflows, deployment workflow, and task-management conventions.
- Use the configured base branch. If none is documented, default to the
  repository's default branch.
- Before creating a new branch from a production base, complete the
  [remote-main preflight](dbt-workflow.md#start-from-current-remote-main).
  Fetch and verify the starting tip before branching, not only at PR creation.
  Do not refresh or rewrite an already-tested feature branch automatically.
- Follow local branch naming. If no convention exists, use a concise
  `feat/`, `fix/`, or `refactor/` name that matches the change type.
- Link the issue, task, specification, or prior PR when one exists. Preserve its
  identifier in the title, body, or commits only where local rules require it.

## Maintain the original PR description

The original PR description is the single reviewer-facing summary. Put the
following there, including results obtained after the PR was opened:

- motivation and scope;
- model grain and contract changes;
- implementation approach;
- development-versus-production validation;
- migration, full-refresh, removal, or downstream BI impact;
- work required before or after merge.

Update the existing validation section for retests, new evidence, changed
row-level results, reporting-impact findings, and authorized deployment checks.
Do not post or refresh separate "Validation update" comments, append a running
update log, or duplicate the same summary in comments. A comment is an
exception only when the user explicitly requests one; a request to validate,
finish, or update the PR does not imply a request for a comment.

Before editing, read the current PR description. Merge the new summary into
the relevant sections, replacing superseded results without erasing material
limitations. Preserve unrelated prose, reviewer edits, template headings,
links, and checklists. Re-read before saving if the body may have changed
since it was fetched, then read back to verify the update. Do not overwrite a
live description with a stale local draft.

If editing is unavailable or unauthorized, report the blocker and provide the
proposed description text; do not substitute a comment. Leave existing
discussion comments untouched unless their editing or removal was requested.

## Gather evidence

Before drafting or updating the description:

1. inspect the full branch diff against the intended base branch;
2. verify that the branch represents one logical piece of work;
3. list changed models, metadata, tests, macros, docs, and downstream contracts;
4. identify the grain and stable key of every changed output;
5. collect separate parse, compile, run, and test outcomes, their exact scopes,
   and meaningful failures, warnings, or skips; keep exact commands internally;
6. collect row-level development-versus-production results, including the
   isolated development schema, model/changed-column allowlist, relation names,
   difference classes based on those columns, and representative keys;
7. collect schema and value evidence only for the selected changed columns.
   Include downstream model/column results only if that validation was
   explicitly requested; otherwise mark it not requested and not performed;
8. identify differences, their demonstrated causes, and acceptance sources
   under [full-validation.md](full-validation.md#accept-or-escalate-differences);
9. for dependency changes, capture actual before/after dbt Docs DAG images
   under [dag-screenshots.md](dag-screenshots.md), with durable image links;
10. identify unresolved release dependencies and exceptional post-merge work,
    such as necessary migrations or backfills. Routine verification and smoke
    testing do not belong in the PR to-do list.

For reporting deprecations, also collect the live API content-impact result
from [reporting-impact.md](reporting-impact.md): affected assets, coverage,
limitations, and agreed migration/retirement actions. This required check is
separate from optional downstream data comparisons.

Keep detailed evidence in the internal developer/tester packets. Gathering
commands, artifact paths, and fingerprints is not an instruction to publish
them. Never paste secrets, credentials, or private customer data.

Treat full validation as complete checks within the recorded changed-column
scope. Unchanged columns and unrequested downstream checks are intentionally
excluded, not incomplete testing, pre-merge tasks, or blockers. Keep any
downstream impact notes separate from claims of executed validation. Do not
classify a required reporting-deprecation API check as unrequested or optional.

For reviewer-facing text, lead with outcomes and actionable limitations. Omit
command transcripts, raw JSON, artifact-file inventories, local paths, commit
and base SHAs, diff hashes, file/content-hash equivalence, and pre-staging
fingerprint narration. Continue verifying code state internally; omit the
verification machinery, not the actual checks. Do not hide those dumps in
collapsible sections. Add technical detail only when explicitly requested or
specifically required by the repository template, and then only what is needed.

## Establish the motivation

Description and Motivation must state both what changes and why the change is
needed. A restatement of moved fields, joins, or preserved behavior is not a
motivation. Use the work ticket or an explicit user explanation; cite the task
and include a concrete rationale sentence in the opening section.

If neither source provides the reason, have the coordinator ask the user early
(ask directly in a standalone invocation). Continue independent preparation
while waiting, but do not invent a business benefit, claim unmeasured savings,
or publish a completed description with motivation missing. Once the user
supplies a rationale, carry it through the handoffs without asking again.

For example, when the user gives the rationale for moving customer attributes,
explain that customer-grain ownership reduces repeated calculation and makes
first-order identifiers easier to understand than placing them at order grain.
Do not reuse that rationale for unrelated work without a supporting source.

## Keep merge tasks actionable

Pre-merge tasks should contain only unresolved release dependencies that need
coordination, such as ordering a dbt release and a companion BI change. For the
customer-field migration example, keep "Coordinate release sequencing" and
explain which side needs the new columns before the old columns disappear.
Do not add routine review, completed validation, generic verification, or
duplicated validation checklists. A real validation blocker remains visible
in Validation of models and must be resolved before a passing handoff; removing
it from the to-do list never waives that requirement.

Post-merge tasks should identify exceptional actions such as a specific
backfill, migration, or required cleanup. Routine smoke testing and normal
production verification are expected operational work and stay unwritten in
this section. Preserve mandatory template headings; leave the section empty
or use "None" when required and no exceptional work remains. Operational work
still follows existing environment permissions and scope.

## Populate the PR description

Use the repository's template as the authoritative structure. Keep its required
headings, remove only sections explicitly marked optional, and mark checklist
items complete only when they are true. A portable analytics PR body commonly
contains:

```markdown
## Description and Motivation

<What changed, followed by a concrete motivation from the ticket/user, and
links to tasks or related PRs.>

## To-do before merge

- [ ] <Only unresolved release coordination; omit if empty and optional.>

## Screenshots

<Real dbt Docs DAG screenshots in separate Before and After dropdowns.>

## Validation of models

<Start with parse/compile/run/test summaries and actual model scope.>

<Changed-column comparison table, followed by material key-metric differences
with reasons and acceptance status. Use the format below.>

<Required reporting-impact findings and unresolved limitations, if any.>

## Changes to existing models

<Grain, columns, semantic contracts, BI changes, compatibility, and any
full-refresh or removal instructions.>

## To-do after merge

- [ ] <Exceptional backfill, migration, or required cleanup only.>

## Checklist

- [ ] The PR is one logical piece of work.
- [ ] Models are materialized appropriately.
- [ ] New models and columns have appropriate tests and documentation.
- [ ] Unique keys or unique column combinations test the declared grain.
```

Use a descriptive title that summarizes the analytics behavior change. State
whether results, grain, schema, history, or downstream contracts change. Do not
describe an intended result as validated unless the cited check actually ran.

## Write the validation section

Keep prose concise without a rigid word or table limit. Use this order:

1. **dbt execution:** quick separate parse, compile, run, and test summaries.
   Identify changed models and, only when explicitly requested, immediate
   downstream candidates. Report failures, skips, and warnings honestly. Say
   all scoped models compiled/ran/tested without failures only when evidence
   supports each claim. Parsing may be project-wide; report its actual scope.
2. **Changed columns:** use tables to show production/development behavior and
   the finding or reason. Group columns only when they share the same result.
   For negligible accepted differences, omit numerical deltas and state that
   the change is negligible and accepted because of the demonstrated reason.
3. **Large key-metric differences:** show production, development, the change,
   the cause, and acceptance status in a table. Preserve significant numbers
   even when accepted. Ask the user if acceptance is uncertain; logic-driven
   metric changes require explicit user acceptance for that behavior/scope.
4. **Reporting impact and limitations:** include required live impact findings
   and actionable unresolved issues. Do not repeat them in generic to-do lists.

Update the repository template's existing validation section; add one only if
it is absent. Use this as a starting point, not a separate comment or mandatory
checklist:

```markdown
## Validation of models

**dbt checks:** Parse: <result>. Compile: <result and scope>.
Run: <result and scope>. Test: <result, scope, failures/warnings/skips>.

**Comparison scope:** <changed columns, common population/window, and key>.

| Changed columns | Production → development | Finding and reason |
| --- | --- | --- |
| <fields sharing one result> | <ownership/type/value behavior> | <match or explained difference> |
| <field with negligible accepted drift> | <qualitative change> | Negligible and accepted because <demonstrated cause>. |

<Include the following only for material key-metric differences.>

| Metric | Production | Development | Change | Reason and acceptance |
| --- | ---: | ---: | ---: | --- |
| <metric> | <value> | <value> | <absolute/relative delta> | <cause; user acceptance or pending decision> |

**Result:** <Pass / Fail / Blocked for the actual scope>.
<Required live reporting impact, asset links, coverage limits, and unresolved
decisions when applicable.>
```

Do not fill this structure with not-applicable rows or invented results. An
empty changed-column allowlist may have no data-comparison table but still
requires reporting impact checks when dependencies are retired. Where useful,
include a single short scope note that downstream data tests were not requested.

Summarize selected-column differences and expected behavior; do not present
matching aggregates as column parity or selected-column equality as whole-row
parity. Use the same comparison scope on both sides. Retain detailed per-row
evidence, sample keys, exact deltas, raw results, and commands internally while
publishing the useful comparison tables above. A durable, access-controlled
evidence link is optional if useful and already available;
do not substitute a list of local JSON files or create a new upload for it.

For example, a retirement with only local checks complete should say that
local checks passed but live reporting impact is still blocked. It must not
say "no pre-PR validation remains" or imply a full pass while saved-content
impact is unknown. A generic "validation complete" request does not warrant
publishing cryptographic/code-provenance details.

## Draft or create the PR

If the ticket/branch already has a PR, update that original PR's description
under the authorized scope; do not create a duplicate PR or a validation
comment. The creation steps below apply only when a new PR is needed.

1. Confirm the current branch, intended base branch, clean staging state, and
   complete diff.
2. Confirm commits are present and related. Do not create or rewrite commits
   unless the user requested that work.
3. Push the branch only with authorization.
4. Write the completed PR body to a temporary file to preserve Markdown and
   avoid shell-escaping errors.
5. Create a draft PR after a current tester `PASS`; draft status does not waive
   that gate. If motivation or screenshots remain unresolved, identify the
   incomplete preparation and do not report the PR as ready for review. Keep
   the PR draft while review or release coordination remains. Create a ready
   PR only when the repository checklist is satisfied and the user requested
   or clearly authorized a ready PR.
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
- Update the original description's relevant sections when validation results,
  scope, migration instructions, or reporting impact change. Keep one current
  validation summary, not a series of appended updates. Do not post a separate
  comment unless the user explicitly requested it.
- After merge, follow the documented deployment workflow and complete the
  production row-level validation and to-do list. Do not deploy or mutate
  production merely because the PR merged.
