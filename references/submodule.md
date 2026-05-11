# Submodule-style repos

Parent repo and child repo have different review targets, so first
identify **which one the PR is for**.

## Telling parent from child

```bash
# Is there a .gitmodules in the current directory?
test -f .gitmodules && echo "parent repo" || echo "regular or child repo"

# List submodules
git config --file .gitmodules --get-regexp path
```

Look at the PR's `files`. If all changes are inside **submodule paths
only**, it's a pointer-bump PR:

```bash
gh pr view <N> --json files --jq '.files[].path'
```

If every path matches a child-repo directory, SHA verification alone is
enough.

## SHA verification flow

For pointer-bump PRs against the parent repo, do **SHA validity
verification** instead of code review.

### 1. Get the new SHA from the PR

```bash
gh pr diff <N>
```

Example output:
```
-Subproject commit aaaaaaa...
+Subproject commit bbbbbbb...
```

`bbbbbbb...` is the new pointer.

### 2. Verify the SHA is merged to main in the child repo

```bash
cd <submodule-path>
git fetch origin main
git merge-base --is-ancestor <new-sha> origin/main \
  && echo "✅ SHA exists on main" \
  || echo "❌ SHA is not on main"
```

### 3. Respond based on the result

**OK**: the parent PR is mergeable. Leave a short comment and finish.

**NG**: the child-repo PR has to merge first. This is a child-before-parent
ordering violation. Tell the user:

```
The SHA <bbbbbbb> this PR points to does not yet exist on
<submodule>'s main branch. Merge the <submodule> PR first, then
re-take the parent pointer:

  cd <submodule>
  git checkout main && git pull
  cd ..
  git submodule update --remote <submodule>
  git add <submodule> && git commit --amend
  git push --force-with-lease
```

## Why not deep-review code from the parent repo

If you deep-review the submodule's contents through the parent-repo PR:

- Findings are not linked to the child repo's PR history (hard to
  track).
- The same finding gets posted twice (parent + child).
- Review ownership becomes ambiguous.

So **the parent repo only checks "is the pointer correct?"**.
Comments on the code itself go on the child repo's PR.

## Aside: reviewing inside the submodule

When reviewing a PR on the child repo, run the normal flow (full steps
3–5). There's nothing submodule-specific to add.

Note: the child repo may have its own `CLAUDE.md` — prefer that one
when working at the child repo's root.
