---
name: finalize
description: 작업 마무리 - 문서 최신화, 커밋, PR/MR 작성을 한 번에 진행합니다. GitHub은 PR(gh), GitLab은 MR(glab)을 자동 선택합니다. 코드 변경 후 마무리 단계에서 사용하세요.
---

# Finalize - 작업 마무리

작업 완료 후 문서 최신화, 커밋, PR/MR 작성을 진행합니다.

## 진행 순서

### 1. 변경 사항 분석
- `git status`로 변경된 파일 확인
- `git diff`로 변경 내용 분석

### 2. Git 플랫폼 감지

`git remote get-url origin`으로 remote URL을 읽어 플랫폼을 판단한다.

| remote URL 패턴 | 플랫폼 | 사용 CLI | 명령어 |
|----------------|--------|---------|--------|
| `github.com` 포함 | GitHub | `gh` | `gh pr create` |
| 그 외 (자체 호스팅 포함) | GitLab | `glab` | `glab mr create` |

- GitHub → 이하 문서에서 "PR" 용어 사용
- GitLab → 이하 문서에서 "MR" 용어 사용

### 3. 문서 최신화

변경 내용을 분석한 후 CLAUDE.md의 PR 전 필수 체크리스트 기준으로 업데이트 대상을 판단한다.

#### 탐색 절차
1. CLAUDE.md를 읽어 문서 업데이트 규칙을 확인한다
2. `.ai/` 디렉토리 구조를 탐색해 존재하는 문서 목록을 파악한다
3. git diff로 변경된 파일/내용을 분석해 아래 표에 따라 업데이트 대상을 결정한다

#### 변경 유형별 문서 업데이트 기준

| 변경 유형 | 업데이트 대상 |
|----------|------------|
| 신규 API 추가 / 엔드포인트 변경 | `README.md` — API 엔드포인트 목록 반영 |
| Controller 추가/수정 | `docs/progress/{app}.md` — 해당 엔드포인트 상태 반영 + `docs/progress/README.md` 진행률 갱신 |
| 아키텍처 / 모듈 구조 변경 | `.ai/architecture/` 해당 문서 반영 |
| 기술 부채 해소 또는 추가 | `docs/tech-debt.md` 반영 |
| Flyway migration 추가 | `.ai/diagrams/erd.md` ERD 반영 |
| Entity 상태 enum 추가/변경 | `.ai/diagrams/{domain}.md` 상태 전이 다이어그램 반영 |
| 위 항목에 해당하지 않는 변경 | 문서 업데이트 생략 |

#### Progress 업데이트 규칙 (`docs/progress/`)

`docs/progress/` 디렉토리가 존재하는 경우에만 수행한다.

1. 변경된 Controller 파일에서 엔드포인트 목록을 추출한다
2. `docs/progress/{app}.md`에서 해당 엔드포인트의 상태를 ✅로 변경하고, 비고란을 갱신한다
3. 엔드포인트가 문서에 없으면 가장 적합한 섹션에 행을 추가한다
4. 엔드포인트 URL이나 HTTP 메서드가 기존 문서와 다르면 실제 코드 기준으로 수정한다
5. 각 섹션의 완료/전체 카운트를 재계산한다
6. `docs/progress/README.md`의 전체 진행률 테이블을 재계산한다
7. 마지막 업데이트 날짜를 오늘로 변경한다

#### 주의
- `.ai/architecture/` 파일들은 아키텍처 결정이 실제로 변경되었을 때만 수정한다
- 존재하지 않는 파일(`docs/tech-debt.md` 등)은 필요 시 새로 생성한다
- `docs/progress/` 디렉토리가 없으면 progress 업데이트를 생략한다

### 4. 커밋 작성
- 프로젝트 Git Conventions 준수:
  - Type: feat, fix, refactor, docs, test, chore
  - 한글로 작성
  - Co-Author 라인 없음
  - 본문은 bullet points

### 5. Push 및 PR/MR 작성
- `git push -u origin <branch>` 실행
- 플랫폼에 따라 PR/MR 생성 (아래 참고)
- base branch: `dev` (이 프로젝트의 기본값 — `main`으로 직접 MR 금지)
- PR/MR이 이미 존재하면 스킵
- push 권한 오류 시 사용자에게 수동 push 안내

#### GitHub (gh)
```bash
gh pr create --base <base-branch> --title "..." --body "..."
```

#### GitLab (glab)
```bash
glab mr create --target-branch <base-branch> --title "..." --description "..."
```

## 커밋 메시지 형식

```
<type>: <subject>

- 변경 사항 1
- 변경 사항 2
```

## PR/MR 본문 형식

```markdown
## Summary
- 변경 사항 요약

## Test plan
- [ ] 테스트 항목
```

## 제외 파일
- `.claude/settings.local.json`
- `logs/`, `*/logs/`
- 기타 로컬 전용 파일

## 실행 흐름

```
1. git status / git diff 분석
2. git remote get-url origin → 플랫폼 감지 (GitHub / GitLab)
3. CLAUDE.md 확인 → 문서 업데이트 규칙 파악
4. .ai/ 디렉토리 탐색 → 존재하는 문서 목록 파악
5. 변경 유형 판단 → 업데이트 대상 문서 결정 및 반영
   - 신규 API → README.md 업데이트
   - 아키텍처 변경 → .ai/architecture/ 해당 문서 업데이트
   - 기술 부채 변경 → docs/tech-debt.md 업데이트
6. git add (제외 파일 제외)
7. git commit
8. git push -u origin <current-branch>
9. [GitHub] gh pr create --base <base-branch>
   [GitLab] glab mr create --target-branch <base-branch>
   (PR/MR 없을 시에만 생성)
10. PR/MR URL 반환
```

## 사용 예시

작업 완료 후:
```
/finalize
```

특정 메시지로 커밋:
```
/finalize Redis 설정 개선 및 테스트 추가
```

## PR/MR 자동 생성 조건
- 현재 브랜치가 `dev`나 `main`이 아닐 때 (feature 브랜치에서만 생성)
- 해당 브랜치의 PR/MR이 아직 없을 때
- push가 성공했을 때

## 브랜치 전략 원칙
- **`dev`에서 직접 작업 금지** — 항상 feature 브랜치에서 작업 후 `dev`로 MR
- MR target은 항상 `dev` (`main`으로 직접 MR 절대 금지)
- 브랜치 생성 예시: `feat/xxx`, `fix/xxx`, `refactor/xxx`
