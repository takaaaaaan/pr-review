# pr-review

Claude Code skill — a general-purpose workflow for reviewing GitHub Pull
Requests from local Claude Code via the `gh` CLI.

It doesn't call the Anthropic API directly, so there is no extra cost —
everything stays within your Claude Code subscription.

## Features

- Search target PRs with `gh pr list`, pick one via `AskUserQuestion`
- Pick a review style from the user:
  - General review
  - Security-focused review
  - Multi-angle review
  - Second opinion (codex)
  - Simplification review
  - Submodule SHA verification
- Fetch and analyze the diff; return a structured result
- Optionally write findings back via `gh pr comment` / `gh api`

## When to use

- "Review this PR", "pull request review", "look at PR #N"
- Listing or inspecting recent / open PRs
- Posting review findings back to GitHub
- Submodule-style repos: validating a parent-repo PR that bumps a
  submodule SHA

## Install

### Via the skills.sh CLI

```bash
npx skills add takaaaaaan/pr-review
```

### Manual install

```bash
git clone https://github.com/takaaaaaan/pr-review.git \
  ~/.claude/skills/pr-review
```

Restart Claude Code, then invoke as `/pr-review`.

## Prerequisites

- `gh` CLI installed and authenticated (`gh auth status` to verify)
- Claude Code

## Layout

```
pr-review/
├── SKILL.md
├── README.md
├── LICENSE
├── references/
│   ├── posting.md         # PR comment posting procedure
│   ├── pr-selection.md    # PR selection flow
│   ├── review-types.md    # Definitions for each review style
│   ├── setup.md           # Initial setup
│   └── submodule.md       # Submodule SHA verification procedure
└── docs/
    ├── README.ja.md       # Japanese
    └── README.ko.md       # Korean
```

## Translations

- 日本語: [docs/README.ja.md](docs/README.ja.md)
- 한국어: [docs/README.ko.md](docs/README.ko.md)

## License

MIT
