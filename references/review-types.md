# Review styles — details

How to execute the style chosen in step 3.

## General review (default)

Generic code quality, bugs, logic errors. Sufficient for most PRs.

Lens:

- Logic correctness (boundary conditions, null/undefined, off-by-one)
- Naming and readability
- Duplication / appropriateness of abstractions
- Roughness in error handling
- Test coverage

How to proceed:

1. Read the whole diff with `gh pr diff <N>`.
2. Drill into larger functions and newly added files first.
3. Where needed, fetch the full file via `gh api` or by Reading it
   directly, and inspect the surrounding code.

## Security-focused

Specialised on auth, authz, input validation, and sensitive data
leakage. Use it for admin-, auth-, and public-API-facing PRs.

Lens:

- Authentication bypass paths (privilege escalation, missing token
  validation)
- Input sanitization (XSS, SQLi, Path Traversal, SSRF)
- Sensitive data in logs or error screens
- Missing encryption, hardcoded secrets
- CORS, CSP, cookie settings (Secure / HttpOnly / SameSite)
- Authorization granularity (IDOR, BOLA)

If a `/security-review` skill is available, invoke it.

## Multi-angle

When you want to cover several lenses in parallel. Use it for large PRs,
refactors, and changes to public APIs.

Lens:

- Comment consistency (mismatch between code and prose)
- Test coverage (do critical paths have tests?)
- Type design (invariants encoded in types, excessive `any`)
- Silent failures (caught and swallowed without logs)
- Simplification opportunities (over-abstraction, dead code)

Use one subsection per lens. If `/pr-review-toolkit:review-pr` is
available, follow its structure.

## Second opinion (codex)

When the call is contested, get an independent review from a different
model (OpenAI Codex CLI). If a `/codex` skill is available, invoke its
review mode.

How to proceed:

1. Run a general review yourself (this session) first.
2. Hand the same diff to `/codex` for an independent review.
3. Compare the two and surface **the items where they disagree** to
   the user.
4. Leave the final call to the user.

Areas where the two often diverge:

- Performance predictions
- Architectural appropriateness
- Extensibility of API design

## Simplification

Suggestions for cutting over-abstraction and redundant code. Most
effective on refactor-style PRs (less so on greenfield features).

Lens:

- Abstraction layers used in exactly one place
- Premature generalisation (YAGNI violation)
- Dead code, unused exports
- Two-paths-for-the-same-thing duplication

If a `/simplify` skill is available, invoke it.

**Note**: simplification suggestions can break functionality easily, so
do not mark them Critical. Present them at Suggestion level and leave
the decision to the user.

## SHA-only

For submodule-style repos when the parent PR just bumps the submodule
pointer. See `submodule.md`.
