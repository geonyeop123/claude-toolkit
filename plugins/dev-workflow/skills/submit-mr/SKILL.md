---
name: submit-mr
description: 작업 브랜치를 커밋하고, PR/MR 을 만들기 **전에** origin/{base}(기본 dev)를 로컬에서 병합해 충돌·파생 파일·마이그레이션 번호 중복을 해소하고, 프로젝트 게이트를 다시 돌린 뒤 push 하고 PR/MR 을 만든 다음 머지 가능 여부까지 확인한다. GitHub(gh)·GitLab(glab) 자동 감지. feature-workflow ⑫-b 가 호출한다. 'MR 올려줘', 'PR 만들어줘', 'push 하고 MR', '머지 요청' 요청 시 사용할 것. 프로젝트에 같은 이름의 스킬(.claude/skills/submit-mr/)이 있으면 그것이 우선이다.
---

# PR/MR 제출 (submit-mr)

## 0. 프로젝트 스킬 우선

**현재 저장소에 `.claude/skills/submit-mr/SKILL.md` 가 있으면 그 스킬을 따르고 여기서 멈춘다.**
프로젝트 스킬은 그 저장소의 게이트·파생 파일·문서 규약을 알고, 이 스킬은 모른다. 두 스킬을 섞어
실행하지 않는다.

## 왜 따로 있는가

PR/MR 을 올린 **뒤에** 충돌을 발견하면 반드시 한 번은 핑퐁이 생긴다 — 리뷰어가 충돌을 알리고, 작성자가
base 를 되병합하고, 파이프라인이 다시 돈다. 이 스킬은 그 되병합을 **PR/MR 생성 전 로컬로 당긴다.**

## 입력

| 값 | 기본값 | 출처 |
|---|---|---|
| `{base}` | `dev` (없으면 원격 기본 브랜치) | feature-workflow ① 에서 정한 값 |
| `{branch}` | 현재 브랜치 | `git branch --show-current`. `{base}`·`main`·`master` 면 **즉시 멈춘다** |
| 호스트 | 자동 | `git remote get-url origin` — `github.com` → `gh`, 그 외 GitLab → `glab` |
| 게이트 | 프로젝트 문서 | 프로젝트 `CLAUDE.md`·`README.md`·CI 설정에서 테스트·린트·빌드 명령을 찾는다. 못 찾으면 사용자에게 묻는다 |
| 이슈 번호 | 브랜치의 `issue-N-` | 있으면 `Closes #N` |

## 1. 작업 트리 정리

문서 최신화(⑫-a) 산출물은 보통 미커밋 상태로 도착한다. **먼저 전부 커밋한다.**

```bash
git status --short          # 의도하지 않은 파일은 넣지 않는다
git add <파일…>
git commit -m "<프로젝트 규약의 메시지>"   # 없으면 Conventional Commits
git status --short          # 비어 있어야 2 절로 간다
```

작업 트리가 비어 있지 않으면 병합하지 않는다 — 병합이 거부되거나, 충돌 해소와 작업 변경이 한 커밋에 섞인다.

## 2. origin 최신화

```bash
git fetch origin
```

**로컬 `{base}` 가 아니라 `origin/{base}` 를 병합한다.** 로컬 base 는 마지막 pull 시점에 멈춰 있어
"충돌 없음" 을 보고하고 PR/MR 에서 충돌한다.

## 3. {base} 선병합

```bash
git merge --no-ff --no-edit origin/{base}
```

- **rebase 가 아니라 merge 다.** push 한 브랜치를 rebase 하면 force push 가 필요하다. 프로젝트가 rebase 를
  명시적으로 요구하고 force push 를 허용할 때만 예외로 한다.
- `Already up to date.` 면 5 절을 건너뛰고 6 절로 간다.

## 4. 충돌·이관 해소

`git diff --name-only --diff-filter=U` 로 충돌 목록을 본다. **충돌이 없어도 아래 "git 이 알리지 않는" 행은
확인한다.**

| 유형 | 해소 |
|---|---|
| 파생 파일 (lockfile·체크섬·생성 코드) | 텍스트 병합하지 않는다. 한쪽을 택한 뒤 **생성 명령으로 다시 만든다**(`npm install`, `./gradlew --write-locks`, 프로젝트 문서의 명령) |
| **마이그레이션 번호 중복** (git 이 알리지 않는다) | Flyway `V*__`·Rails `db/migrate` 등 번호 기반 마이그레이션은 파일명이 달라 충돌이 나지 않고 앱만 뜨지 않는다. 병합 뒤 번호 중복을 직접 검사해 **우리 쪽** 번호를 올린다 — base 쪽은 배포됐을 수 있다 |
| 목록·색인·이력 문서 | 양쪽 행을 모두 살린다. 프로젝트가 그 파일을 폐지했으면(스텁·README 안내) 안내대로 옮긴다 |
| 코드 | 양쪽 의도를 확인해 해소하고 해당 모듈 테스트를 돌린다. 모르면 추측하지 말고 사용자에게 묻는다 |

해소 뒤 `git add` → `git commit --no-edit` 으로 병합 커밋을 완성한다(`--no-edit` 가 없으면 에디터가 열려 에이전트 셸에서 멈춘다).

## 5. 게이트 재실행

3 절에서 병합 커밋이 생겼으면 — 충돌이 없었더라도 — 프로젝트 게이트(테스트·린트·빌드)를 **다시** 돌린다.
통과해야 push 한다.

- exit code 로 판정한다. `| tail` 같은 파이프를 붙이지 않는다 — 실패를 exit 0 으로 바꾼다.
- 병합 전에 초록이었어도 생략하지 않는다. 텍스트 충돌 없는 의미 충돌(한쪽이 지운 함수를 다른 쪽이 호출)이
  여기서만 드러난다.

## 6. push 와 PR/MR 생성

```bash
git push -u origin {branch}
```

훅이 걸리면 우회하지 말고 원인을 고친다. PR/MR 이 **아직 없을 때만** 만든다.

```bash
# GitHub
gh pr list --head {branch} --base {base} --state open   # 이미 있으면 생성하지 않는다
gh pr create --base {base} --head {branch} --title "<제목>" --body-file <본문 파일> [--draft]
# GitLab
glab mr list --source-branch {branch} --target-branch {base}
glab mr create --target-branch {base} --source-branch {branch} --title "<제목>" --description "<본문>" --yes [--draft]
```

- **이슈 close 필수:** 이슈 컨텍스트면 본문 첫 줄 `Closes #N`. `Refs`·`Related` 로 대체하지 않는다.
- 미완 작업이 남았으면 Draft 로 만들고 `Closes` 를 보류한다.

## 7. MR 뒤 충돌 확인

```bash
gh pr view {branch} --json mergeable,mergeStateStatus     # GitHub
glab mr view {branch} --output json                        # GitLab: detailed_merge_status / has_conflicts
```

- 계산 중(`UNKNOWN`, GitLab `checking`·`unchecked`)이면 30 초 간격으로 최대 3 회 재조회한다.
- 충돌(`CONFLICTING`·`DIRTY`, GitLab `conflict`)이면 **2 절부터 다시** 돈다. push 만 하고 새 PR/MR 을
  만들지 않는다.
- **이 확인은 생성 시점의 스냅샷이다.** 머지 큐가 없으면 이후 base 가 전진해 다시 충돌할 수 있다.
  리뷰가 길어졌으면 머지 직전에 2 절부터 다시 부른다.

## 금지

| 하지 않는 것 | 이유 |
|---|---|
| base·main 에 직접 push | 작업 브랜치에서 PR/MR 로만 들어간다 |
| 임의 force push | 리뷰 중인 커밋이 사라진다 |
| 로컬 base 병합 | 낡은 기준으로 "충돌 없음" 을 보고한다 |
| 파생 파일 텍스트 병합 | 두 값을 섞은 결과는 어느 쪽에도 맞지 않는다 |
| 병합 뒤 게이트 생략 | 의미 충돌은 텍스트 충돌이 없어도 생긴다 |
