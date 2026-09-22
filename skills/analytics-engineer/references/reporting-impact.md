# Live Reporting Impact for Deprecations

## Contents

- [Establish the live dependency scope](#establish-the-live-dependency-scope)
- [Looker example](#looker-example)
- [Decide and report](#decide-and-report)

Read this when retiring, removing, or incompatibly renaming a reporting
dependency: a warehouse model or column used by BI, semantic resource, LookML
view or Explore, dataset, or report used by other saved content.

This check is required for deprecation work, even under a light/static
validation scope. It is a narrow exception to opt-in downstream validation:
inspect live content dependencies, not downstream data values. Do not rebuild
downstream dbt models, execute dashboard queries, or profile unchanged columns
unless separately requested.

## Establish the live dependency scope

The developer identifies the retiring objects, their reporting-layer mappings,
and any intended replacements. The tester owns the live impact check before
certifying the retirement. The coordinator routes unresolved impact back to
development or asks the user for the necessary access or migration decision.

Use the reporting tool's available connector, API, or supported SDK with
existing authorized credentials. Discover the correct tenant, API version, and
permissions from project guidance or connection metadata; do not invent
endpoints or print secrets. Repository reference searches and local parsers
supplement live checks but cannot establish that hosted content is unaffected.

For each relevant reporting platform:

- Enumerate saved content and dependencies through its API. Follow pagination
  and include accessible personal and shared content, not just checked-in or
  recently viewed dashboards. A title search alone is not a dependency check.
- Match exact dataset/model/Explore/field identities and aliases. Trace saved
  queries, filters, calculated fields, tiles, embedded reports, and linked
  saved content as supported. Include dependent schedules and alerts where
  exposed; disclose unsupported asset types or inaccessible scopes.
- Use native content/dependency validation when available. Verify whether it
  evaluated current production or the actual proposed development change.
  A clean result against pre-removal production does not prove that removing
  a still-valid dependency is safe.
- Record impacted asset names, IDs/URLs, broken dependency, and required
  migration or retirement. Include owner and usage only when available and
  useful for coordination; low usage is not proof that content is disposable.
- Record the check time, covered projects/folders, pagination completion,
  permission limits, and baseline/candidate identity in the tester packet.
  Refresh the check if the candidate or relevant saved content changes.

## Looker example

Use the installed API/SDK version and repository connection workflow.

1. Map retired objects to the LookML model, Explores, view aliases, and fields
   used by saved queries. Include join/filter dependencies, not just selected
   display fields.
2. Inventory saved Looks, user-defined dashboards and tiles, and LookML
   dashboards. Looker's dashboard search excludes LookML dashboards, which
   have a separate [search method](https://docs.cloud.google.com/looker/docs/reference/looker-api/latest/methods/Dashboard/search_lookml_dashboards).
   Resolve linked Looks and merged-query components when encountered.
3. Use the [Validate Content API](https://docs.cloud.google.com/looker/docs/reference/looker-api/latest/methods/Content/content_validation)
   and map errors back to the retiring objects. Compare baseline errors with
   candidate errors when an isolated development workspace contains the exact
   proposed LookML; distinguish existing breakage from new breakage.
4. Check validation coverage: project/folder scoping and permissions can hide
   relevant content. Folder-scoped validation excludes schedules and alerts;
   project scoping may miss content for a model removed from that project.
   Supplement those gaps with inventory or appropriately scoped validation.
   See [Content Validator behavior and limits](https://docs.cloud.google.com/looker/docs/content-validation).

Preserve concurrent Looker development work. Verify workspace and branch
before candidate validation; do not overwrite another task's development
state or deploy the branch to make validation possible. If safe candidate
validation is unavailable, use current saved-query dependency mappings to
identify predicted impact and clearly label what was not verified.

## Decide and report

- No dependencies found with adequate live coverage: report no impacted
  content found **within that coverage**, not a universal guarantee.
- Dependencies found: identify the affected assets and return unresolved
  breaking dependencies for migration or an explicit retirement decision.
  An intended, authorized removal of content is not an unexpected regression;
  record the agreed action and timing without labeling it unaffected.
- API unavailable, incomplete visibility, or unresolvable references: report
  impact as unknown/blocked with the exact missing access or evidence. Continue
  independent local checks, but do not issue an overall retirement pass while
  this required check is unresolved. Do not relabel it as unrequested
  downstream testing. Ask for access or an explicit scope/risk decision;
  an accepted limitation remains a limitation, not a passed live check.

Keep production content read-only. Do not use content-validator replace/remove
actions, alter dashboards or schedules, delete reports, notify owners, or
deploy changes without authorization for those actions. The request to assess
impact does not authorize remediation in a reporting service.

The deployer summarizes impact counts, important asset links, coverage limits,
and required actions in the original PR description, updating its existing
sections rather than posting a validation comment. Keep raw API responses and
inventories in access-controlled handoff evidence.
