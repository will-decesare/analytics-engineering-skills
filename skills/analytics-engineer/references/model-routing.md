# Model routing

Use these defaults for the coordinated analytics workflow unless the user or
applicable repository instructions explicitly select different models.

| Assignment | Model | Reasoning effort |
| --- | --- | --- |
| Coordinator planning, architecture, and escalation decisions | `gpt-6-astra` | `high` |
| Developer implementation and repair | `gpt-6-astra` | `high` |
| Independent tester: test design, interpretation, and final gate | `gpt-6-astra` | `high` |
| Routine evidence collection from a defined plan | `gpt-5.6-terra` | `medium` |
| Deployer: Git/PR preparation from verified handoffs | `gpt-5.6-luna` | `medium` |

## Select models at launch

The coordinator explicitly supplies the model and reasoning effort when
starting each specialist or evidence worker. These are execution settings,
not fields to add to a skill's `agents/openai.yaml`. With collaboration
`spawn_agent`, pass `model` and `reasoning_effort` and use `fork_turns="none"`:
full-history forks inherit the parent settings and cannot apply overrides.
Supply a self-contained assignment with the relevant skill path, repository
instructions, ticket criteria, current handoff, fingerprint, scope, authority,
and required output. Link detailed artifacts instead of copying full logs or
conversation history.

A skill cannot change the model of its already-running caller. Start the
coordinator on the strong default above when setting up the workflow. If a
skill is invoked directly, retain the caller's selected model and disclose
any difference from these defaults; do not create a redundant agent merely
to switch the caller. When model selection or a requested model is unavailable,
use the inherited model, report the fallback, and do not claim cost savings.
Keep the sequential-role fallback when separate agents are unavailable.

## Delegate routine evidence, retain judgment

The developer or tester may ask the coordinator for a bounded evidence worker
when independent collection can run alongside useful work. The coordinator
starts that worker and routes its result back to the requesting specialist.
Batch related evidence work; keep tiny or immediately blocking checks local
when delegation would only add overhead.

Suitable assignments include extracting results from existing dbt artifacts,
collecting logs or CI state, enumerating live reporting content for specified
identifiers, and executing exact, already-reviewed read-only comparison
queries. The requesting specialist defines sources, fingerprint, column and
time scope, commands or queries, and expected output before delegation.
Keep schema setup and materialization with the tester. Evidence workers do
not write implementation, warehouse data, Git/PR state, or hosted content;
local evidence files must use their own output paths.

Workers return commands/API calls, source timestamps, coverage, results, and
artifact paths to the coordinator, which forwards them to the specialist.
They do not design business rules, change selectors or queries to obtain a
pass, explain away differences, or issue a tester verdict. The strong tester
checks freshness and coverage, investigates differences, and owns the final
`PASS`, `FAIL`, or `BLOCKED`, including reporting-retirement clearance.

## Escalate without weakening checks

A cheaper worker returns its partial evidence and exact unresolved issue
when requirements are ambiguous, results conflict, comparison keys fail,
differences are unexplained, or reporting coverage is uncertain. The
coordinator routes interpretation to the strong tester and implementation
repairs to the strong developer. Do not repeat speculative cheap-model
attempts or broaden scope to get an answer.

The deployer returns stale fingerprints or missing passes for revalidation.
If Git/PR preparation itself needs complex reasoning, replace it with a
deployer using the strong model/effort from the table and the same handoffs
and authority; stop the earlier worker before another writer proceeds.
Missing access or authorization remains a blocker regardless of model.

Record requested model/effort, actual settings when exposed, fallbacks, and
escalations in internal handoffs. Model choice never reduces required tests,
independence, fingerprint checks, reporting-impact coverage, or authorization.
