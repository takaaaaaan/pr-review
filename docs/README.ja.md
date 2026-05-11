# pr-review

Claude Code skill — ローカル Claude Code から `gh` CLI 経由で GitHub Pull Request をレビューするための汎用ワークフロー。

Anthropic API を直接叩かないので、追加課金なしで Claude Code 定額枠の中で完結します。

## 機能

- `gh pr list` で対象 PR を検索 → `AskUserQuestion` で選択
- レビュー方式をユーザー選択:
  - 一般レビュー
  - セキュリティ重視レビュー
  - 多角レビュー
  - 第二意見（codex）
  - 簡素化レビュー
  - サブモジュール SHA 検証
- diff を取得・分析し、構造化された結果を返す
- 必要に応じて `gh pr comment` / `gh api` で PR にコメント投稿

## いつ使うか

- 「PR をレビュー」「pull request review」「PR #N を見て」と依頼されたとき
- open PR を一覧したい、最近の PR を確認したいとき
- レビュー結果を GitHub に書き戻したいとき
- サブモジュール構成リポジトリで親リポのサブモジュール SHA 更新 PR を検証したいとき

## インストール

### skills.sh CLI 経由

```bash
npx skills add takaaaaaan/pr-review
```

### 手動インストール

```bash
git clone https://github.com/takaaaaaan/pr-review.git \
  ~/.claude/skills/pr-review
```

Claude Code を再起動すると `/pr-review` として利用可能になります。

## 前提条件

- `gh` CLI がインストール済み・認証済み (`gh auth status` で確認)
- Claude Code

## 構成

```
pr-review/
├── SKILL.md
├── README.md
├── LICENSE
├── references/
│   ├── posting.md         # PR へのコメント投稿手順
│   ├── pr-selection.md    # PR 選択フロー
│   ├── review-types.md    # 各レビュー方式の定義
│   ├── setup.md           # 初期セットアップ
│   └── submodule.md       # サブモジュール SHA 検証手順
└── docs/
    ├── README.ja.md       # 日本語（このファイル）
    └── README.ko.md       # 韓国語
```

## ライセンス

MIT
