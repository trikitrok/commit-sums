---
name: commit-itinerary
description: Narrate a supplied sequence of git commits as an itinerary, summarizing each transition, its intent evidence, and its practical impact without modifying the repository. Use when the user asks for a commit itinerary, history walkthrough, or narrative across a list of commit hashes.
---

# Commit Itinerary

Turn a user-supplied, ordered list of at least two Git revisions into a concise
itinerary: each adjacent pair is a leg, and the full itinerary explains the
overall engineering direction. Do not pull, fetch, reset, switch branches, or
otherwise modify the repository.

Treat the supplied order as deliberate. State the direction as
`commit 1 → commit 2 → …`. If the commits are not in a linear ancestor chain,
explain where the itinerary crosses branches rather than silently reordering
them.

## Invocation examples

- `Create a commit itinerary for the last three commits.`
- `Walk me through a1b2c3d → e4f5g6h → i7j8k9l.`
- `Tell the history across these commits: a1b2c3d, e4f5g6h, i7j8k9l.`
- `Map the engineering journey from release-2026-09 through release-2026-10.`
- `$commit-itinerary a1b2c3d e4f5g6h i7j8k9l`

## Validate the itinerary

Run these commands from the repository the user is working in:

```sh
git rev-parse --show-toplevel
git rev-parse --verify <revision>^{commit}
git status --short
```

- If the directory is not a Git repository, say so.
- If a revision does not resolve to exactly one commit, identify it and ask for
  a valid replacement; do not guess.
- Require at least two revisions. For a single commit, use `commitsum`
  instead.
- Record the author, date, and subject for each supplied commit before writing
  the report.

## Build every leg with diffsum

For every adjacent pair `<from>, <to>`, apply the `diffsum` workflow to
`<from>..<to>`. Inspect its commit range, file/status changes, patch, intent
evidence, and practical impact. Do not merely summarize the target commit:
each leg describes the difference between its two itinerary stops.

For each leg, check whether `<from>` is an ancestor of `<to>`:

```sh
git merge-base --is-ancestor <from> <to>
```

- For a linear leg, report the commits and themes introduced from `<from>` to
  `<to>`.
- For a cross-branch leg, compare the two snapshots and use
  `git log <from>...<to>` to identify commits unique to each side. Label it as
  a cross-branch comparison; do not frame it as a progression.
- Reuse PR context already gathered for a commit across later legs. When GitHub
  CLI is available and the remote is on GitHub, use its commit-to-PR lookup as
  described by `diffsum`.

For long ranges, prioritize production logic, public APIs, schemas,
configuration, dependencies, and behavior-describing tests. Group related
commits within a leg rather than enumerating low-value formatting or generated
file churn.

## Synthesize the journey

Compare the first and last stops directly to identify the overall result, then
reconcile that with the legs. Highlight the most consequential additions,
removals, reversals, and decisions. Explain the apparent overall intention
only to the extent supported by PRs, commit messages, and code/tests.

Clearly distinguish evidence from inference: use wording such as "PR #123
says…" for stated rationale and "Likely…" for inference. Say "Reason unclear"
when evidence is inadequate. Flag dependencies, migrations, configuration or
environment changes, API compatibility changes, removed/renamed files, and
build or test workflow changes. If the worktree is dirty, note affected-file
overlap but do not change it.

## Report

Lead with the destination and use this shape:

```md
**Itinerary: `<short first>` → `<short last>`** (<n> stops, <m> legs)

<2–3 sentences on the journey's overall direction, most important change, and stated or likely intent.>

## Overall destination
- **<Theme>**: <end-to-end outcome>. <Evidence or clearly labeled inference.>

## Itinerary
### 1. `<short from>` → `<short to>` — <leg title>
<What changed in this leg, why, and the practical effect.>

### 2. …

## Practical impact
- <Action required or "No action needed">

## Worth a look
- <Cross-branch leg, risk, breaking change, reversal, or unclear rationale>
```

Give every requested leg its own itinerary step, but keep each step compact.
Order details by impact within a leg. Link commits and PRs when the remote
supports GitHub links. Omit empty sections except **Practical impact**, where
you write "No action needed".
