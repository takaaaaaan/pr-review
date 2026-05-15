---
name: pr-review
description: |
  GitHub PR review workflow via the `gh` CLI from local Claude Code. User picks
  a PR from `gh pr list`, then a review style (general / security / multi-angle
  / second-opinion / simplification / submodule SHA verification). Fetches and
  analyzes the diff, returns structured results, and optionally posts comments
  back to the PR. No Anthropic API cost — stays within Claude Code subscription.

  Use when: reviewing an existing PR ("review this PR", "look at PR #N"),
  listing/inspecting recent PRs, posting findings back to GitHub via
  `gh pr comment`/`gh api`, validating a parent-repo PR that bumps a submodule
  SHA, or when the repo/PR is ambiguous and candidates must be listed first.
---

# PR Review Skill

A general-purpose skill for running an end-to-end PR review workflow from
local Claude Code using the `gh` CLI.

## Design principles (why this structure)

**All user confirmations go through the `AskUserQuestion` tool.**
Plain-text "Should I do X?" prompts that wait for a free-form reply are
forbidden. Three reasons:

- Explicit options let the user see at a glance what choices exist.
- The button UI means the user does not have to type a response.
- "Recommended" markers and per-option `description` text communicate
  both the recommendation and the reason in one place.

Anywhere this skill defers to the user (PR selection, review style,
post / no-post, install gh or not, etc.) it must call `AskUserQuestion`.
The only exception is when the request explicitly fixes the answer to a
single choice.

**Review outcomes depend heavily on "what to review" and "what lens to use".**
If the user keeps both in their head, results drift run to run, so this
skill makes both selections explicit through `AskUserQuestion`.

**Avoid Anthropic API billing — never call it directly.**
The Claude Code session itself does the review work. `gh` is used only to
fetch information and to write findings back to the PR.

**Don't trespass on human responsibility.**
PR approve / merge / irreversible comment posting always requires a final
explicit confirmation, even under auto mode. That confirmation also goes
through `AskUserQuestion`.

---

## Execution flow

Once the request comes in, always proceed in this order. Each step has
a corresponding reference file with the details — the SKILL.md keeps only
the skeleton and the decision points; the meat lives under `references/`.

```
[1] Environment check    → is gh CLI ready?
[2] PR selection         → list candidates and have the user pick
[3] Review style         → have the user pick the lens
[4] Run the review       → fetch diff + analyze + structure
[5] Present + post?      → post / fix it yourself / no share, user picks
```

---

## Step 1: Environment check

Run `gh --version && gh auth status`.

- **Not installed** — read `references/setup.md`, then use
  **`AskUserQuestion`** to offer three options: "install / show command only /
  abort". Never ask in plain text. If "install" is picked, detect the OS and
  run `winget` / `brew` / `apt` etc., then continue to the auth flow.
  **Do not start an install on your own.**
- **Not authenticated** — guide the user to run `gh auth login`. This needs
  browser auth and is interactive, so Claude Code can't run it for them.
- **Both OK** — proceed to the next step.

---

## Step 2: PR selection

If the user did not specify a concrete PR number, or asked something broad
like "list open PRs", fetch candidates first:

```bash
gh pr list --state open --json number,title,author,headRefName,updatedAt --limit 30
```

For submodule-style repos, either `cd` into the submodule the user
mentioned, or use `gh --repo owner/name`.

When there are multiple candidates, **always use `AskUserQuestion`** to let
the user pick. Don't ask "which PR?" in plain text. See the design
principles for the reason.

- **1–4 candidates** → one option per PR
  (label = `#N title (author)`, description = `updatedAt / branch name`).
- **5+ candidates** → put the 4 most recent in options, use `Other` to
  accept a free-form PR number.
- **0 candidates** → ask with `AskUserQuestion`:
  "search with `--state all` / specify another repo / abort".

If the request already includes an explicit PR number ("review PR #5"),
skip the selection step. Otherwise, do not skip it.

Details: `references/pr-selection.md`.

---

## Step 3: Review style

A review can take many angles. Unless the request unambiguously fixes the
style, **always ask via `AskUserQuestion`**. Don't use plain text. This is
not just "which style is best?" — it forces the user to verbalize what
they want from the review.

`AskUserQuestion` allows up to 4 options per question, so split it like
this.

**Standard 4-choice menu** (covers most cases):

| label                        | description                                                              |
| ---------------------------- | ------------------------------------------------------------------------ |
| General review (Recommended) | General code quality, bugs, logic errors. Sufficient for most PRs.       |
| Security-focused             | Auth, authz, input validation, sensitive-data leakage.                   |
| Multi-angle                  | Comment consistency, test coverage, type design, silent failures.        |
| Other                        | Second opinion (codex) / simplification / SHA-only — pick on next step.  |

If "Other" is selected, follow up with a second `AskUserQuestion`:

| label                | description                                                          |
| -------------------- | -------------------------------------------------------------------- |
| Second opinion (codex) | Independent review by a different model (important / contested PRs). |
| Simplification       | Spot over-abstraction and redundant code.                            |
| SHA-only             | Just verify the submodule SHA exists on main.                        |
| Abort                | Exit without reviewing.                                              |

If the request itself gives a clear signal ("look at security", "quick
pass", "just SHA check"), infer and proceed. Only ask when uncertain.

Execution details for each style: `references/review-types.md`.

---

## Step 4: Run the review

Fetch the diff according to the chosen style:

```bash
gh pr view <N> --json title,body,author,headRefName,baseRefName,files,additions,deletions
gh pr diff <N>
```

**Handling a large diff**

- If the diff is too big to review at full depth, break it down per file.
- Use `gh pr diff <N> -- <path>` for a single file.
- Still too big? Ask the user: "overall summary + Critical excerpts" or
  "deep dive per file"?

**Handling missing context**

- When the diff alone is insufficient, fetch the surrounding file via
  `gh api` or by reading it directly, then judge.
- If the repo has a `CLAUDE.md`, read it and fold its conventions into
  the review lens.

---

## Step 5: Format the result + decide whether to post

The review output **must** use the structure below. The
Critical / Important / Suggestion 3-tier scheme lets the user instantly
gauge how heavy each comment is.

```markdown
## 🤖 PR #<number> Review

### 🔴 Critical (must fix before merge)
- <file>:<line> — <comment>

### 🟡 Important (should fix)
- <file>:<line> — <comment>

### 🟢 Suggestion (nice to have)
- <file>:<line> — <comment>

### ✅ Strengths
- <good points>

### 📋 Recommended actions
1. ...
```

**Don't overuse Critical.** Reserve it for things that genuinely block
the merge (security hole, data corruption, build breakage). Loose
standards drain the tier of meaning.

Once formatted, **don't post anything on your own**. **Always use
`AskUserQuestion`** to offer these 4 options. No plain-text prompts.
Posting is irreversible, so do not skip this confirmation even in auto mode.

| label                              | description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| Fix it myself (Recommended)        | Edit/Write the file directly. Fastest path.                  |
| Post as summary comment            | `gh pr comment <N> --body-file <tmp>`.                       |
| Post as inline review comments     | `gh api repos/.../pulls/<N>/reviews` (create a review).      |
| Don't post, don't fix              | Just read the result and stop.                               |

**When the PR is your own, "Fix it myself" is usually fastest** — keep it
as Recommended at the top. For someone else's PR or when the team needs
visibility, pick a posting option.

Details: `references/posting.md`.

---

## Things you must not do

- **Post a PR comment without confirmation** — visible to the team and
  irreversible. Final confirmation is required even in auto mode.
- **Call the Anthropic API directly** — defeats the point of the skill.
  Do not use `curl https://api.anthropic.com/...`.
- **Run `gh pr review --approve` automatically** — approval is a human
  responsibility; a machine doesn't stand in for it.
- **Edit PR title / body silently** — `gh pr edit` only when explicitly
  asked.
- **Overuse Critical** — keep the bar high. When in doubt, demote to
  Important.

---

## Behavior in submodule-style repos

Parent and child repos require different reviews, so first identify
**which one the PR targets**. Ask via `AskUserQuestion` if needed.

| Target repo               | Primary review lens                                       |
| ------------------------- | --------------------------------------------------------- |
| Child repo (the code)     | Normal review (run steps 3–5 in full).                    |
| Parent repo (pointer bump) | SHA verification only.                                    |

Details: `references/submodule.md`.

---

## Output style & language localization

Always respond in the language the user is speaking — including
**every `AskUserQuestion` `question`, `header`, `label`, and `description`
field**. The English snippets shown in this SKILL.md and its
`references/*.md` are templates, not literal outputs. Translate them to
the user's language at call time.

Priority order when deciding the language:
1. An explicit language directive in the project's `CLAUDE.md`.
2. The language of the user's most recent message.
3. English (fallback).

Other rules:

- The review body itself follows the chosen language. Variable names,
  function names, file paths, CLI flags, and SHAs stay verbatim.
- Code blocks always include a language tag (` ```ts ` etc.).
- Be short and focused. Concrete `file:line` findings beat long prose.

---

## When to read which reference file

This SKILL.md is the skeleton. Read these as needed:

| File                               | When to read it                                                  |
| ---------------------------------- | ---------------------------------------------------------------- |
| `references/setup.md`              | Step 1 reveals `gh` is not ready.                                |
| `references/pr-selection.md`       | Step 2 needs complex filtering.                                  |
| `references/review-types.md`       | Step 3 needs the detailed procedure for a specific style.        |
| `references/posting.md`            | Step 5 needs `gh api` for inline review comments.                |
| `references/submodule.md`          | You're verifying a submodule SHA bump.                           |
