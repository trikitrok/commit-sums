---
name: commit-logbook
description: Present an ordered sequence of git commits as a nautical Markdown logbook, with each transition, evidence for intent, and practical impact. Use when the user asks for a logbook itinerary, nautical commit history, or ship's-log view of commits.
---

# Commit Logbook

Turn a commit journey into a concise nautical-style Markdown logbook. The user
must provide at least two Git revisions, or a clear scoped request such as
"the last three commits." Do not pull, fetch, reset, switch branches, or
otherwise modify the repository.

## Invocation examples

- `Create a commit logbook for the last three commits.`
- `Log this journey: a1b2c3d → e4f5g6h → i7j8k9l.`
- `Tell this commit history as a ship's log: a1b2c3d, e4f5g6h, i7j8k9l.`
- `Give me a nautical Markdown logbook for release-2026-09 through release-2026-10.`
- `$commit-logbook the last five commits`

## Delegate the analysis

Apply the `commit-itinerary` skill to the ordered list of commits. Reuse its
validation, per-leg `diffsum` analysis, intent-evidence standards, handling of
non-linear histories, practical-impact checks, and caution about dirty
worktrees. Do not repeat or weaken that investigation merely because this
skill changes the presentation.

Preserve the supplied order. The first commit is the departure point; every
adjacent pair is a leg. Retain its distinction between evidence (PRs, commit
messages, code, tests) and inference (clearly marked "Likely").

## Render the logbook

Present the results as rendered Markdown, without an outer code fence unless
the user asks for copyable Markdown source. Use a restrained nautical tone:
clear navigation metaphors are welcome, but never replace technical facts,
hide risks, or invent maritime details such as a branch name, reviewer, or PR.

Use this shape, adapting it to the evidence:

```md
# ⚓ Ship's Log — <short journey title>

**Voyage:** `<short first>` → `<short last>`  
**Stops:** <n> | **Legs:** <m> | **Date range:** <range>  
**Author(s):** <names>  
**Course:** <linear history, or clearly named cross-branch comparison>.

> *<1–2 sentence technical overview in logbook voice.>*

## Destination reached

- **<Theme>** — <end-to-end technical result>. <Evidence or clearly labeled inference.>

---

## First leg — <short thematic title>

### `<short from>` → `<short to>`
**Log entry:** *<commit subject or theme>*

<What changed in this leg.>

**Evidence of intent:** <stated reason, or "Reason unclear">.

**Likely purpose:** <inference, only if supported>.

**Notable cargo:**

- <important addition, removal, API/configuration change, or test change>

---

## Orders for the crew

- <required local action, or "No action needed">

## Charted hazards

- <risk, breaking change, cross-branch caveat, missing PR context, or unclear rationale>
```

- Give every requested leg its own `##` section, even when the journey is
  long; keep individual legs compact and prioritize impact.
- Include the **Orders for the crew** section with "No action needed" when
  applicable.
- Include **Charted hazards** only when there is a real concern or a material
  evidence limitation.
- Use simple Markdown, headings, rules, and the anchor symbol; do not add HTML,
  images, CSS, or brand-specific visual assets.
