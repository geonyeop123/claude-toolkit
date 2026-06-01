---
name: check-plugin-compat
description: "superpowers 플러그인 업데이트 후 기존 커스텀 스킬과의 호환성을 검증한다. 'plugin 업데이트했어', 'superpowers 버전 올렸어', '플러그인 호환성 체크', '스킬 깨진 거 없어?' 등 플러그인 업데이트 직후 또는 스킬 호출이 실패했을 때 반드시 사용할 것."
---

# Plugin Compatibility Check

superpowers 플러그인 업데이트 후, 커스텀 스킬(feature-workflow 등)이 참조하는 플러그인 스킬의 호환성을 검증한다.

## 실행 절차

### Step 1: 현재 플러그인 상태 확인

```bash
# 설치된 superpowers 버전 확인
ls ~/.claude-work/plugins/cache/claude-plugins-official/superpowers/

# 현재 활성 버전의 스킬 목록
ls ~/.claude-work/plugins/cache/claude-plugins-official/superpowers/*/skills/
```

### Step 2: 의존 스킬 존재 확인

feature-workflow가 의존하는 superpowers 스킬 3개가 여전히 존재하는지 확인한다.

| 의존 스킬 | 사용 위치 | 검증 방법 |
|----------|----------|----------|
| `superpowers:brainstorming` | feature-workflow ① | 스킬 디렉터리 + SKILL.md 존재 확인 |
| `superpowers:writing-plans` | feature-workflow ③ | 스킬 디렉터리 + SKILL.md 존재 확인 |
| `superpowers:subagent-driven-development` | feature-workflow ⑧ | 스킬 디렉터리 + SKILL.md 존재 확인 |

```bash
PLUGIN_DIR=$(ls -d ~/.claude-work/plugins/cache/claude-plugins-official/superpowers/*/ | sort -V | tail -1)
echo "Active version: $PLUGIN_DIR"

for skill in brainstorming writing-plans subagent-driven-development; do
  if [ -f "${PLUGIN_DIR}skills/${skill}/SKILL.md" ]; then
    echo "✅ $skill — exists"
  else
    echo "❌ $skill — MISSING"
  fi
done
```

**❌가 하나라도 있으면:** 해당 스킬이 이름 변경되었거나 제거됨. Step 5로 이동.

### Step 3: 경로 의존성 확인

feature-workflow와 CLAUDE.md가 참조하는 파일 경로가 플러그인에서 여전히 사용되는지 확인한다.

```bash
PLUGIN_DIR=$(ls -d ~/.claude-work/plugins/cache/claude-plugins-official/superpowers/*/ | sort -V | tail -1)

echo "=== docs/superpowers/specs/ 경로 참조 ==="
grep -r "docs/superpowers/specs/" "${PLUGIN_DIR}skills/" --include="*.md" -l

echo "=== docs/superpowers/plans/ 경로 참조 ==="
grep -r "docs/superpowers/plans/" "${PLUGIN_DIR}skills/" --include="*.md" -l
```

**경로가 변경되었으면:**
- feature-workflow의 ③④ 단계 경로 수정 필요
- CLAUDE.md PR 체크리스트 경로 수정 필요
- tech-docs 에이전트의 매트릭스 경로 수정 필요

### Step 4: 동작 변경 감지

각 의존 스킬의 SKILL.md를 읽어 핵심 동작이 변경되었는지 확인한다.

**brainstorming** — 확인 항목:
- 출력 파일 경로가 `docs/superpowers/specs/YYYY-MM-DD-*-design.md` 형식을 유지하는가
- "Phase" 기반 구조가 유지되는가

```bash
PLUGIN_DIR=$(ls -d ~/.claude-work/plugins/cache/claude-plugins-official/superpowers/*/ | sort -V | tail -1)
grep -n "docs/superpowers" "${PLUGIN_DIR}skills/brainstorming/SKILL.md"
```

**writing-plans** — 확인 항목:
- 출력 파일 경로가 `docs/superpowers/plans/YYYY-MM-DD-*.md` 형식을 유지하는가
- 태스크 분해 형식(체크박스)이 유지되는가

```bash
grep -n "docs/superpowers" "${PLUGIN_DIR}skills/writing-plans/SKILL.md"
```

**subagent-driven-development** — 확인 항목:
- 플랜 파일을 입력으로 받는 형식이 유지되는가
- worktree 사용 여부가 변경되었는가

```bash
grep -n "plan\|worktree\|task" "${PLUGIN_DIR}skills/subagent-driven-development/SKILL.md" | head -20
```

### Step 5: 결과 보고

```markdown
## Plugin Compatibility Report

**Plugin version:** {version}
**Check date:** {date}

### 스킬 존재 확인
- [ ] superpowers:brainstorming — ✅ / ❌
- [ ] superpowers:writing-plans — ✅ / ❌
- [ ] superpowers:subagent-driven-development — ✅ / ❌

### 경로 호환성
- [ ] docs/superpowers/specs/ — 유지 / 변경됨 → {새 경로}
- [ ] docs/superpowers/plans/ — 유지 / 변경됨 → {새 경로}

### 동작 변경
- [ ] brainstorming 출력 형식 — 유지 / 변경됨 → {변경 내용}
- [ ] writing-plans 출력 형식 — 유지 / 변경됨 → {변경 내용}
- [ ] subagent-driven-development 입력 형식 — 유지 / 변경됨 → {변경 내용}

### 필요한 조치
- (조치 목록 또는 "조치 불필요")
```

### Step 6: 자동 수정 (문제 발견 시)

문제가 발견되면 영향받는 커스텀 스킬을 수정한다.

| 문제 | 수정 대상 | 수정 내용 |
|------|----------|----------|
| 스킬 이름 변경 | `feature-workflow/SKILL.md` | 이전 이름 → 새 이름으로 치환 |
| 출력 경로 변경 | `feature-workflow/SKILL.md`, `CLAUDE.md`, `.claude/agents/tech-docs.md` | 이전 경로 → 새 경로로 치환 |
| 스킬 제거 | `feature-workflow/SKILL.md` | 해당 단계를 대체 스킬로 교체하거나 수동 단계로 전환 |
| 입력 형식 변경 | `feature-workflow/SKILL.md` | 호출 방식 업데이트 |

**수정 후 반드시 확인:**
1. feature-workflow의 전체 단계 흐름이 논리적으로 유효한가
2. 경로 참조가 모두 일관되는가 (CLAUDE.md ↔ feature-workflow ↔ tech-docs)

## 영향받는 커스텀 파일 목록

플러그인 업데이트 시 점검해야 할 파일 전체 목록:

```
~/.claude-work/skills/feature-workflow/SKILL.md     — superpowers 스킬 3개 호출
프로젝트/CLAUDE.md                                    — docs/superpowers/ 경로 참조
프로젝트/.claude/agents/tech-docs.md                  — docs/superpowers/ 경로 의존 (간접)
프로젝트/docs/agent-workflow.md                       — 워크플로우 문서
```
