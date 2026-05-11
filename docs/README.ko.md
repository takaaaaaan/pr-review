# pr-review

Claude Code skill — 로컬 Claude Code 에서 `gh` CLI 를 통해 GitHub Pull Request 를 리뷰하기 위한 범용 워크플로우.

Anthropic API 를 직접 호출하지 않으므로 추가 과금 없이 Claude Code 정액제 범위 내에서 완결됩니다.

## 기능

- `gh pr list` 로 대상 PR 을 검색 → `AskUserQuestion` 으로 선택
- 리뷰 방식 사용자 선택:
  - 일반 리뷰
  - 보안 중심 리뷰
  - 다각도 리뷰
  - 제2 의견 (codex)
  - 단순화 리뷰
  - 서브모듈 SHA 검증
- diff 를 가져와 분석하고 구조화된 결과를 반환
- 필요 시 `gh pr comment` / `gh api` 로 PR 에 코멘트 게시

## 언제 사용하나

- "PR 을 리뷰", "pull request review", "PR #N 을 봐줘" 라고 요청받았을 때
- open PR 을 나열하거나 최근 PR 을 확인하고 싶을 때
- 리뷰 결과를 GitHub 에 작성하고 싶을 때
- 서브모듈 구성 저장소에서 부모 저장소의 서브모듈 SHA 갱신 PR 을 검증하고 싶을 때

## 설치

### skills.sh CLI 사용

```bash
npx skills add takaaaaaan/pr-review
```

### 수동 설치

```bash
git clone https://github.com/takaaaaaan/pr-review.git \
  ~/.claude/skills/pr-review
```

Claude Code 를 재시작하면 `/pr-review` 로 호출할 수 있습니다.

## 사전 요구사항

- `gh` CLI 설치 및 인증 완료 (`gh auth status` 로 확인)
- Claude Code

## 구성

```
pr-review/
├── SKILL.md
├── README.md
├── LICENSE
├── references/
│   ├── posting.md         # PR 코멘트 게시 절차
│   ├── pr-selection.md    # PR 선택 흐름
│   ├── review-types.md    # 각 리뷰 방식 정의
│   ├── setup.md           # 초기 셋업
│   └── submodule.md       # 서브모듈 SHA 검증 절차
└── docs/
    ├── README.ja.md       # 일본어
    └── README.ko.md       # 한국어 (이 파일)
```

## 라이선스

MIT
