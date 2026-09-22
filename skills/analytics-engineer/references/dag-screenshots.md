# Before and after dbt DAG screenshots

Use this workflow when model dependencies change or the user requests DAG
screenshots. The developer preserves the baseline; the tester captures and
checks it; the deployer embeds the images in the PR. If no graph changes exist,
omit an optional screenshot section or state that the DAG is unchanged when
the template requires the section. Do not claim screenshots are unavailable
without attempting the native dbt Docs workflow.

## Generate and serve both states

1. Identify the actual before/base revision and the tested after fingerprint.
   Use separate checkouts and task-owned docs target/log directories so docs
   generation cannot overwrite validation artifacts or the other docs state.
   Preserve an active or dirty checkout; do not reset it for a screenshot.
2. In each checkout, use the project's installed dbt version and supported
   profile to run `dbt docs generate`, then `dbt docs serve` only after
   generation succeeds: the intended sequence is
   `dbt docs generate && dbt docs serve`. Pass the appropriate isolated target
   and log paths and an available port to each invocation using that version's
   supported flags. Use the same generated target path for generation and
   serving, and inspect the server output for its actual local URL.
3. Use separate server sessions/ports for before and after, or capture each
   sequentially. Keep warehouse access within the task's existing authority;
   generating documentation does not authorize production builds or writes.
   Do not substitute after-state artifacts into the before site or treat a
   docs generation compile as evidence of successful model runs/tests.
4. Open each served site with the available browser tooling, navigate to the
   lineage/DAG view, and frame the affected models and their relevant edges.
   Use matching filters, orientation, viewport, and zoom where possible so the
   dependency changes are easy to compare. Wait for graph rendering before
   capturing actual browser screenshots.
5. Inspect both images: model labels must be readable, changed dependencies
   visible, and baseline/candidate identities correct. A screenshot of the
   docs landing page, a hand-drawn diagram, or a generated mockup is not DAG
   evidence. If a field move leaves graph edges unchanged, explain that limit;
   do not fabricate a visual change.

The [dbt Docs command reference](https://docs.getdbt.com/reference/commands/cmd-docs)
describes generation and serving; flags and defaults vary by installed version.

## Publish and verify

Save the two images and capture identities in the internal tester packet.
Use a repository-supported, access-appropriate image attachment mechanism
within the authorized PR workflow. Embed durable URLs reachable by reviewers;
local filesystem paths or localhost links will not render for them. Do not
commit generated docs/catalogs, expose internal docs publicly, or upload to a
new external service merely to obtain an image URL.

Use separate dropdowns in the repository's Screenshots section:

```markdown
<details>
<summary>Before</summary>

![DAG before the change](<before-image-url>)

</details>

<details>
<summary>After</summary>

![DAG after the change](<after-image-url>)

</details>
```

Expand both dropdowns in the rendered PR and verify the images load and remain
legible. Stop only the documentation servers started for this capture once
finished. If generation, browser capture, or attachment is blocked, return the
exact failure and any available local images to the coordinator; request the
missing capability and leave screenshot preparation explicitly incomplete.
Do not silently substitute "did not screenshot" or mark the PR complete.
