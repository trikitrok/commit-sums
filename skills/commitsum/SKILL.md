---
name: commitsum
description: Summarize the changes introduced by a specified git commit hash, including what changed, who made it, and the stated or likely reason. Use when the user says "commitsum", "summarize this commit", or asks what a commit hash did.
---

# Commitsum

Explain one specified commit as a short, evidence-based briefing. The user must
provide a revision that resolves to exactly one commit. Do not pull, fetch,
reset, switch branches, or alter the worktree.

## Invocation examples

- `Commitsum a1b2c3d.`
- `Summarize commit a1b2c3d.`
- `What did commit a1b2c3d change?`
- `Explain HEAD as a commit briefing.`
- `$commitsum release-2026-10`

## Identify the commit

Run these commands from the repository the user is working in:

```sh
git rev-parse --show-toplevel
git rev-parse --verify <revision>^{commit}
git show --no-ext-diff --format=fuller --no-patch <commit>
git diff-tree --no-commit-id --name-status -r <commit>
```

- If the directory is not a Git repository, say so.
- If the revision does not resolve to a single commit, report that and ask for
  a valid hash; do not guess.
- Record the commit's author, authored date, subject, and parent count.

## Delegate change analysis to diffsum

For an ordinary commit with one first parent, apply the `diffsum` workflow to
`<commit>^..<commit>`. Use its evidence and impact analysis rather than
repeating its inspection rules. Present the resulting comparison as the change
introduced by `<commit>`.

- For a merge commit, compare its first parent to the merge commit and
  explicitly state in the report that this is a **first-parent integration
  comparison**. Do not imply this is the same as every branch contribution.
- For an initial/root commit, explicitly state in the report that this is a
  **root commit with no parent**, so `diffsum` cannot be applied. Use
  `git show --no-ext-diff --format=fuller <commit>` and treat the full tree
  introduction as the change. Apply `diffsum`'s standards for intent evidence,
  practical impact, and uncertainty.

Retain the commit-specific metadata and frame the shared analysis with this
report shape:

```md
**Commit `<short sha>` — `<subject>`** by <author> (<date>)

<1–3 sentences on the change and its overall purpose.>

## What changed
- **<Theme or component>**: <concrete change>. <Stated or likely reason.>

## Impact
- <Action required, compatibility concern, or "No action needed">

## Worth a look
- <Risk, surprising behavior, revert, or unclear rationale>
```

Omit empty sections except **Impact**, where you write "No action needed".
Link to the commit and associated PR when the remote supports GitHub links.
