---
name: diffsum
description: Summarize the changes between two specified git commits, with evidence for intent and practical impact, without modifying the repository. Use when the user says "diffsum", "summarize the diff between commits", or asks what changed from one commit hash to another.
---

# Diffsum: Compare and Summarize

Explain the changes from a base revision to a target revision as a concise,
evidence-based briefing. The user must provide two Git revisions. Do not pull,
fetch, reset, switch branches, or otherwise modify the repository.

## Invocation examples

- `Diffsum a1b2c3d e4f5g6h.`
- `What changed from a1b2c3d to e4f5g6h?`
- `Compare HEAD~5 with HEAD.`
- `Give me the intent and practical impact of release-2026-09..release-2026-10.`
- `$diffsum main feature/new-routing`

## Resolve and inspect the range

Run these commands from the repository the user is working in:

```sh
git rev-parse --show-toplevel
git rev-parse --verify <base>^{commit}
git rev-parse --verify <target>^{commit}
git merge-base --is-ancestor <base> <target>
git log --no-merges --reverse --format='%h%x09%an%x09%ad%x09%s' --date=short <base>..<target>
git shortlog -sn --no-merges <base>..<target>
git diff --stat <base> <target>
git diff --name-status <base> <target>
git diff --no-ext-diff <base> <target>
```

- If the directory is not a Git repository, say so.
- If either revision cannot be resolved to exactly one commit, report it and
  ask for valid hashes; do not guess.
- State the direction explicitly: the report describes `<base>..<target>`.
- If the base is not an ancestor of the target, say that this is a
  cross-branch comparison rather than a linear history range. Compare the two
  snapshots, and use `git log <base>...<target>` to identify commits unique to
  each side; do not imply that one is a direct successor of the other.
- For large changes, inspect the most consequential patch sections in detail:
  production logic, public APIs, schemas, configuration, dependencies, and
  tests that describe changed behavior.

## Establish intent

Read the commits in the range using `git show --stat <sha>` and the relevant
parts of each patch. Group related commits by feature or pull request rather
than reciting commit-by-commit history.

When GitHub CLI is available and the remote is on GitHub, look up pull-request
context once per relevant commit:

```sh
gh api repos/{owner}/{repo}/commits/<commit>/pulls --jq '.[] | {number, title, body, user: .user.login}'
```

Use PR titles, descriptions, and linked issues as the best evidence for why.
Then use commit messages and code/tests. Distinguish stated rationale (for
example, "PR #123 says…") from inference ("Likely…"). Use "Reason unclear"
when the evidence does not support a conclusion.

## Flag practical impact

Call out new or changed dependencies, migrations, environment variables or
configuration, breaking or renamed APIs, removed or renamed files, and changes
to build or test commands. If the worktree is dirty, report any overlap with
the files changed by the comparison, but do not modify it.

## Report

Lead with the overall outcome and use this structure:

```md
**`<short base>` → `<short target>`**: <n> commits, <m> people

<1–3 sentences on the dominant themes and whether the commits form a linear range.>

## What changed
- **<Theme or component>**: <concrete change>. <Stated or likely reason>. (<people>, PR #<n>)

## Who did what
- **<Name>** (<n> commits): <one line on their focus>

## Impact
- <Action required or "No action needed">

## Worth a look
- <Risk, compatibility concern, surprising deletion, revert, or unclear rationale>
```

Order themes by impact. Omit empty sections except **Impact**, where you write
"No action needed". For more than about 30 commits, summarize the five most
important themes and consolidate the remainder. Link commits and PRs when the
remote supports GitHub links.
