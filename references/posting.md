# Posting comments to a PR

Concrete commands and gotchas when writing review results back to a PR.

## Core rule

Writing to a PR is **irreversible** and visible to the team. Even under
auto mode, get one final user confirmation before posting.

## A. Post as a summary comment

The simplest path: write the whole review as a single PR comment.

```bash
# Use a temp file to dodge shell escaping issues
TMP=$(mktemp --suffix=.md)
cat > "$TMP" <<'EOF'
## 🤖 PR Review

### 🔴 Critical
- ...

### 🟡 Important
- ...
EOF

gh pr comment <N> --body-file "$TMP"
rm "$TMP"
```

PowerShell version:

```powershell
$tmp = New-TemporaryFile
@"
## 🤖 PR Review
...
"@ | Set-Content -Path $tmp -Encoding utf8
gh pr comment <N> --body-file $tmp
Remove-Item $tmp
```

## B. Post as an inline review

Create a review containing inline comments on specific lines. The CLI
doesn't cover this directly, so call the GitHub API via `gh api`.

### Review with a single inline comment

```bash
gh api repos/:owner/:repo/pulls/<N>/reviews \
  -X POST \
  -F body="Overall good, with a few notes" \
  -F event=COMMENT \
  -f "comments[][path]=src/auth.ts" \
  -F "comments[][line]=42" \
  -f "comments[][body]=Use crypto.timingSafeEqual for password compare"
```

### Multiple inline comments

Send a JSON payload in one shot:

```bash
cat > /tmp/review.json <<'EOF'
{
  "body": "Overall review: ...",
  "event": "COMMENT",
  "comments": [
    {"path": "src/auth.ts", "line": 42, "body": "..."},
    {"path": "src/db.ts",   "line": 88, "body": "..."}
  ]
}
EOF

gh api repos/:owner/:repo/pulls/<N>/reviews \
  -X POST \
  --input /tmp/review.json
```

### Choosing the `event` field

| event             | Meaning                              |
| ----------------- | ------------------------------------ |
| `COMMENT`         | Comment only. No approve / decline.  |
| `APPROVE`         | Approve. **The skill must not pick this.** |
| `REQUEST_CHANGES` | Request changes. Be careful.         |

**Never auto-run `APPROVE` from the skill.** Approval is the human's call.

## C. Don't post — fix it yourself

If it's your own PR and the fix is small, this is the fastest path.

1. Edit/Write the affected file.
2. Check that the tests still pass.
3. `git add && git commit && git push`.
4. The PR updates automatically — no comment posting needed.

This flow keeps the "address Critical findings, re-review" cycle tight.

## Handling errors

### `HTTP 403`

Insufficient scope. Run
`gh auth refresh -h github.com -s repo`.

### `HTTP 422 - Validation Failed`

The `line` is probably outside the diff hunk context. `line` must be a
**line number that appears in the diff**. Check `additions` via
`gh pr view <N> --json files`, or pass `side: "RIGHT"` explicitly.

### A pending review is in the way

A `Pending review` blocks creating a new one. Delete it:

```bash
gh api repos/:owner/:repo/pulls/<N>/reviews --jq '.[] | select(.state=="PENDING") | .id' \
  | xargs -I {} gh api repos/:owner/:repo/pulls/<N>/reviews/{} -X DELETE
```

## Concrete pre-post confirmation

**Always use the `AskUserQuestion` tool.** Don't ask "OK to post?" in
plain text.

```
question: Post this review to the PR? (irreversible)
header: post review

[Show the body preview in the previous assistant message]

options:
  - label: "Post (Recommended)"
    description: "Write to the PR as a summary comment"
  - label: "Edit then post"
    description: "Trim to Critical-only first, then post"
  - label: "Post as inline"
    description: "Post line-by-line as inline review comments"
  - label: "Don't post"
    description: "Fix it yourself, or hold off"
```

Show the review body preview in the assistant message immediately
before the `AskUserQuestion` call. Keep `description` short; put long
explanations in the body.
