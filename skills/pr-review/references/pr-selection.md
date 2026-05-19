# PR selection — detailed procedure

When the user did not specify a PR number, narrow the target here.

## Baseline: list open PRs

```bash
gh pr list --state open --json number,title,author,headRefName,updatedAt --limit 30
```

## Filter by author

"Only my PRs" / "PRs by this person":

```bash
gh pr list --author @me --state open --json number,title,headRefName
gh pr list --author <username> --state open
```

## Filter by label

```bash
gh pr list --label "needs-review" --state open
gh pr list --label "wip" --state open
```

## Exclude drafts

```bash
gh pr list --state open --search "draft:false"
```

## Target a different repo

For submodule layouts or cross-project references:

```bash
gh pr list --repo owner/repo --state open
```

## AskUserQuestion presentation patterns

### 4 or fewer candidates

One option per PR:

```
question: Which PR do you want to review?
options:
  - label: "#42 Fix auth flow (taro)"
  - label: "#41 Bump nginx (jiro)"
  - label: "#40 Tidy logs (saburo)"
```

### 5 or more candidates

Show the 4 most recent and fall back to `Other`:

```
question: Which PR do you want to review? (showing 4 most recent)
options:
  - label: "#42 Fix auth flow"
  - label: "#41 Bump nginx"
  - label: "#40 Tidy logs"
  - label: "#39 Uptime Kuma config"
```

If the user types a different PR number via `Other`, run `gh pr view`
with that number.

### 0 candidates

```
question: No open PRs found. What now?
options:
  - label: "Include closed"
    description: re-search with --state all
  - label: "Different repo"
    description: specify --repo owner/name
  - label: "Abort"
```

## Reviewing multiple PRs at once

If the user says "review all of them", use `AskUserQuestion` with
`multiSelect: true` to pick targets. Cap selection at 4 and tell them
the rest will be done in a follow-up batch.

Loop through each PR and **finish with an aggregate summary**.
Per-PR details can be collapsed.

## Notes

- `gh pr list` defaults to open only — pass `--state` explicitly for
  closed / merged.
- `--limit` defaults to 30 — bump it if you need more.
- In large repos, listing is slow. Consider `--limit 10` for the first
  pass.
