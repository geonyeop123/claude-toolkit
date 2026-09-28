# claude-toolkit

geonyeop123 개인 **Claude Code 툴킷**. 프로젝트에 종속되지 않는 **범용 개발 워크플로우 스킬/에이전트**를 모은 개인 plugin marketplace.
새 컴퓨터에서 두 줄로 설치하면 어떤 프로젝트에서든 동일한 개발 파이프라인을 사용할 수 있다.

## 설치 (신규 PC)

```bash
# 1) 이 repo를 marketplace로 등록
claude plugin marketplace add geonyeop123/claude-toolkit

# 2) dev-workflow 플러그인 설치
claude plugin install dev-workflow@claude-toolkit
```

업데이트 / 제거:

```bash
claude plugin marketplace update claude-toolkit   # 최신 동기화
claude plugin uninstall dev-workflow@claude-toolkit
```

## 사전 요구사항 (Prerequisites)

스킬에 따라 외부 도구가 필요하다. 없는 항목은 해당 스킬만 동작하지 않는다.

| 의존 | 필요로 하는 스킬 | 설치 |
|------|----------------|------|
| **superpowers 플러그인** | `feature-workflow`, `check-plugin-compat`, `codex-subagent-driven-development` | superpowers marketplace 설치 |
| **openai-codex 플러그인 + codex CLI** | `codex-code-review`, `codex-spec-review`, `codex-subagent-driven-development` | codex 플러그인 설치 + 로컬 codex CLI |
| **gh CLI** (GitHub) | `finalize`, `submit-mr`, `issue` | `gh auth login` |
| **glab CLI** (GitLab) | `finalize`, `submit-mr`, `issue` | GitLab 사용 시 |

## 포함된 스킬 (10)

| 스킬 | 용도 |
|------|------|
| `feature-workflow` | 기능 개발 전체 라이프사이클 오케스트레이션(브랜치/worktree→브레인스토밍→스펙→플랜→구현→리뷰→마무리). 단계별 위임 모델·탐색 위임·self-healing 포함 |
| `harness` | 신규 도메인/프로젝트에 전문 에이전트 + 스킬을 생성하는 **메타 generator** |
| `planning-with-files` | Manus 스타일 파일 기반 플래닝(task_plan/findings/progress) |
| `add-convention` | 새 코딩 컨벤션/규칙을 프로젝트 문서에 추가 |
| `codex-spec-review` | 로컬 codex CLI로 스펙/플랜 문서 리뷰(ACCEPT/REBUT 루프) |
| `codex-code-review` | 로컬 codex CLI로 브랜치/PR 코드 리뷰 |
| `codex-subagent-driven-development` | codex를 implementer로 두고 task 단위 구현 + Claude 리뷰 |
| `issue` | GitHub/GitLab 이슈 등록(host 자동 감지, 부모/자식 분할) |
| `finalize` | 문서 최신화 + 커밋 + PR/MR 생성(gh/glab 자동 선택) |
| `submit-mr` | PR/MR 전 `origin/{base}` 선병합 → 충돌·파생 파일 해소 → 게이트 재실행 → push → PR/MR 생성 → 머지 가능 확인. feature-workflow ⑫-b 가 호출. 프로젝트 `.claude/skills/submit-mr/` 우선 |
| `check-plugin-compat` | superpowers 플러그인 업데이트 후 커스텀 스킬 호환성 검증 |

## 포함된 에이전트 (2)

| 에이전트 | 용도 |
|---------|------|
| `tech-docs-design` | 설계 이해가 필요한 문서(ERD·상태 다이어그램·아키텍처·tech-debt) 갱신 (Sonnet) |
| `tech-docs-mechanical` | 기계적 문서(진행률·카운트·체크박스) 갱신 (Haiku) |

> 두 에이전트는 `feature-workflow` ⑪-a 단계가 호출한다.

## 구조

```
claude-toolkit/
├── .claude-plugin/marketplace.json   # marketplace 정의
└── plugins/
    └── dev-workflow/
        ├── .claude-plugin/plugin.json
        ├── skills/<10 skills>/
        └── agents/tech-docs-{design,mechanical}.md
```

## 제외된 것 (의도적)

- **프로젝트 종속 스킬/에이전트**(premium-spread·backend·career-coach 등의 도메인 orchestrator/implementer 등)는 포함하지 않는다. 전역 설치 시 무관한 프로젝트에서 오발화하기 때문. 이들은 각 프로젝트 repo에서 관리하고, 새 프로젝트는 `harness` 스킬로 생성한다.
- `weekly-report`, `karpathy-guidelines`: 다른 스킬 의존성 없음을 확인 후 범위에서 제외(필요 시 개별 추가 가능).
- `내전기록`, `gnsm-dto-convention`(dto-convention): 개인/특정 프로젝트 전용이라 제외.

## 갱신 워크플로우

원본은 `~/.claude/skills/` (및 `~/.claude/commands/finalize`)에서 관리한다.
스킬을 수정한 뒤 이 repo로 동기화 → commit/push → 각 PC에서 `claude plugin marketplace update`.
