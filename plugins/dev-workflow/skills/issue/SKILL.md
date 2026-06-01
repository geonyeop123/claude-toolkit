---
name: issue
description: Use when the user invokes /issue to register a GitHub or GitLab issue for a fix, feat, refactor, chore, or docs task — auto-detects host from git remote, brainstorms scope, splits parent/child when the work is too large, and creates the issue via gh or glab CLI. Registration only; consuming the issue happens in a separate session.
---

# /issue — 이슈 등록 스킬

## 개요

`/issue` 슬래시 커맨드로 호출. 현재 저장소가 GitHub인지 GitLab인지 git remote에서 자동 감지하여 `gh` 또는 `glab` CLI로 이슈를 생성한다. 브레인스토밍을 거쳐 본문을 구성하고, 작업이 크면 **최상위 이슈 + 하위 이슈**로 분리한다.

**본 스킬은 "등록"까지만 책임진다.** 추후 세션에서 등록된 이슈를 consume하여 작업을 시작하는 흐름은 본 스킬의 범위가 아니다.

## 호출 형태

| 형태 | 동작 |
|------|------|
| `/issue` | 풀 브레인스토밍 모드. 한 줄 설명부터 사용자에게 물어본다. |
| `/issue <한 줄 설명>` | 한 줄 설명을 시작점으로 풀 브레인스토밍. |
| `/issue --simple <한 줄 설명>` | 간단형. 제목/타입/배경/완료조건만 받고 즉시 등록. |
| `/issue --simple` | 간단형. 사용자에게 핵심 4개만 물어보고 등록. |

`--simple` 플래그가 없으면 항상 풀 모드. 사용자가 명시적으로 간단형을 선택했을 때만 축소 흐름을 탄다.

## 워크플로

```
1. Host 감지 (git remote)        → gh or glab 결정
2. CLI 인증 상태 확인              → 미인증 시 안내 후 중단
3. 프로젝트 컨텍스트 수집           → CLAUDE.md 브레인스토밍 체크리스트, 최근 이슈 라벨
4. 브레인스토밍                    → 풀 모드 or 간단 모드
5. 분량 판단                       → 태스크 8개 초과/도메인 2개 이상이면 parent/child 제안
6. 본문 draft 작성 → 사용자 승인    → 수정 사이클
7. 이슈 등록 (gh/glab)             → 번호와 URL 출력
```

## 1. Host 자동 감지

```bash
REMOTE=$(git remote get-url origin 2>/dev/null)
case "$REMOTE" in
  *github.com*)  HOST=github ; CLI=gh ;;
  *gitlab*)      HOST=gitlab ; CLI=glab ;;
  *)             # 사용자에게 호스트 확인 후 진행 ;;
esac
```

remote가 없는 저장소면 즉시 중단하고 사용자에게 알린다.

## 2. CLI 인증 확인

| CLI | 인증 확인 | 실패 시 안내 |
|-----|-----------|--------------|
| `gh` | `gh auth status` | `gh auth login` 안내 후 중단 |
| `glab` | `glab auth status` | `glab auth login` 또는 토큰 설정 안내 후 중단 |

GitLab 사내 인스턴스(예: `gitlab.saycore.kr`)는 remote URL의 호스트를 `glab` host 설정과 매칭하여 사용한다.

## 3. 프로젝트 컨텍스트 수집

이슈를 등록하기 전에 다음을 살펴 본문 품질을 높인다:

- `CLAUDE.md` — 프로젝트별 브레인스토밍 체크리스트가 있으면 **이 스킬의 기본 체크리스트보다 우선 적용**한다.
- 최근 이슈 라벨 — `gh issue list --limit 20` / `glab issue list --per-page 20` 으로 라벨 컨벤션을 파악.
- 진행 중인 마일스톤/에픽이 있는지 (있다면 사용자에게 묶을지 묻기).

## 4. 브레인스토밍

### 4-1. 풀 모드 체크리스트

프로젝트 CLAUDE.md에 브레인스토밍 체크리스트가 있으면 그것을 사용한다. 없으면 아래 기본 체크리스트:

- [ ] **타입** — fix / feat / refactor / chore / docs / perf / test 중 하나
- [ ] **제목** — `[type] 한 줄 요약` 형태, 50자 이내 권장
- [ ] **배경/문제** — 왜 필요한가? 현재 어떤 상황인가?
- [ ] **스코프** — 무엇을 포함하고, 무엇을 제외하는가?
- [ ] **요구사항/기대 동작** — 구체적 변경 사항
- [ ] **수용 기준 (Acceptance Criteria)** — 완료를 판단하는 객관적 조건
- [ ] **영향 범위** — 어느 모듈/도메인/API가 바뀌는가
- [ ] **테스트 전략** — 단위/슬라이스/통합/E2E 중 무엇이 필요한가
- [ ] **위험/주의사항** — 호환성, 데이터 마이그레이션, 보안, 성능 등
- [ ] **참조 링크** — 관련 PR, 문서, 외부 spec, 이전 이슈
- [ ] **라벨** — type, priority, domain 라벨 (프로젝트 컨벤션 확인)

### 4-2. 간단 모드 (`--simple`)

다음 4개만 물어보고 등록:

1. **타입** (fix/feat/refactor/...)
2. **제목**
3. **배경 한 줄**
4. **완료 조건 한두 줄**

라벨은 타입에 해당하는 것만 자동 부착. 본문은 미니멀 템플릿(아래 §6 참조)으로 생성.

## 5. 최상위/하위 이슈 분리 판단

다음 중 하나라도 해당하면 **parent + children 구조를 제안**한다:

- 예상 태스크 수가 **8개 초과** (브레인스토밍 중 작업 단위가 8개를 넘어가는 시점)
- 영향 도메인/모듈이 **2개 이상**이고 독립 배포·테스트가 가능
- 마이그레이션이 단계적으로 진행되어야 하는 경우 (V1 스키마 변경 → V2 데이터 백필 → V3 컬럼 제거 등)
- API/스키마/이벤트 등 **계약 변경**과 **소비 측 적용**이 모두 포함된 경우

제안 형식:
```
이 작업은 parent 1개 + child N개로 나누는 게 좋을 것 같습니다.
- Parent: [feat] 전체 목표 한 줄
  - Child 1: ...
  - Child 2: ...
사용자 승인 후, parent 먼저 등록 → 번호 확보 → children 본문에 "Parent: #N" 추가하여 등록.
```

사용자가 "그냥 하나로 등록"이라고 하면 단일 이슈로 진행.

## 6. 본문 템플릿

### 풀 모드

```markdown
## 배경
<왜 이 작업이 필요한가>

## 스코프
**포함**
- ...

**제외**
- ...

## 요구사항
- ...

## 수용 기준 (Acceptance Criteria)
- [ ] ...
- [ ] ...

## 영향 범위
- 모듈/도메인: ...
- API 변경 여부: ...
- 마이그레이션 필요 여부: ...

## 테스트 전략
- ...

## 위험 / 주의사항
- ...

## 참조
- 관련 PR/MR: ...
- 문서: ...
- 이전 이슈: ...
```

### 간단 모드 (`--simple`)

```markdown
## 배경
<한 줄>

## 완료 조건
- [ ] ...
```

### Parent / Child 추가 섹션

Parent 이슈 본문 끝에:
```markdown
## 하위 이슈
- [ ] #<number1> — child 제목
- [ ] #<number2> — child 제목
```

Child 이슈 본문 끝에:
```markdown
## Parent
- #<parent-number>
```

## 7. 이슈 등록 명령

**GitHub:**
```bash
gh issue create \
  --title "<title>" \
  --body-file <(cat <<'EOF'
<body>
EOF
) \
  --label "<labels-comma-separated>"
```

**GitLab:**
```bash
glab issue create \
  --title "<title>" \
  --description "$(cat body.md)" \
  --label "<labels-comma-separated>"
```

> **GitLab은 description 인자에 stdin/heredoc 직접 주입 시 escape 이슈가 잦다.** 본문은 항상 **임시 파일에 먼저 기록**하고 `$(cat body.md)` 로 주입한다. 등록 후 임시 파일은 삭제.

등록이 성공하면 반환된 이슈 번호와 URL을 사용자에게 보여준다. 실패하면 stderr 그대로 출력하고 멈춘다 (자동 재시도 금지).

## 8. 라벨 컨벤션

- 타입 라벨 필수: `type:fix`, `type:feat`, `type:refactor`, `type:chore`, `type:docs`, `type:perf`, `type:test`
- 프로젝트 라벨 컨벤션이 다르면 (예: `bug`, `enhancement`) 기존 라벨을 따른다 — §3 조회 결과 우선.
- Priority/severity 라벨이 있으면 사용자에게 물어본다.

## 9. 절대 금지

- **이슈 등록 전 사용자 최종 승인 없이 자동 등록 금지.** Draft를 보여주고 "이대로 등록할까요?" 확인.
- **풀 모드에서 체크리스트 항목을 임의 생략 금지.** 사용자가 "스킵" 의사를 명시한 경우만 비워둔다.
- **외부 시스템에 등록되는 작업이므로** (이슈는 팀에 공유됨) 추측 가능한 정보는 모두 사용자에게 확인 후 기재.
- **GitHub와 GitLab을 혼동하지 않는다.** remote 감지 결과를 사용자에게 한 번 알린 뒤 진행.

## 10. 출력 형식

등록 완료 후 사용자에게 보여줄 내용:

```
✅ 이슈 등록 완료
- Host: GitLab (gitlab.saycore.kr)
- Issue: #142 [feat] 비밀번호 정책 변경
- URL: https://gitlab.saycore.kr/.../-/issues/142
- 라벨: type:feat, domain:auth
```

Parent/child 구조면 parent와 모든 child의 번호·URL을 표로 보여준다.

## Common Mistakes

| 실수 | 올바른 처리 |
|------|------------|
| body를 인자 문자열로 직접 주입 → 줄바꿈/특수문자 깨짐 | 임시 파일에 기록 후 `--body-file` / `$(cat ...)` 사용 |
| remote 미확인 채로 `gh` 호출 → GitLab 저장소에서 실패 | §1 호스트 감지 먼저 |
| CLAUDE.md 체크리스트 무시하고 기본 템플릿 사용 | §3 컨텍스트 수집을 먼저 수행 |
| 큰 기능을 단일 이슈로 등록 | §5 기준으로 parent/child 분리 제안 |
| 사용자 승인 없이 등록 | §9 — 항상 draft 승인 후 등록 |
