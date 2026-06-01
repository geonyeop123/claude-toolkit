---
name: add-convention
description: Use when a user wants to add a new coding convention or rule to the project documentation. Analyzes existing docs, decides the right file, and adds the content directly.
user-invocable: true
allowed-tools:
  - Read
  - Edit
  - Write
  - Glob
  - Grep
---

# Add Convention

사용자가 컨벤션/규칙을 추가하고 싶을 때 올바른 문서 파일을 판단해 직접 추가한다.

## 프로세스

```
1. 컨벤션 내용 파악
2. 문서 구조 확인 (필요 시)
3. 파일 결정
4. 내용 추가
5. 결과 보고
```

---

## Step 1: 컨벤션 내용 파악 및 검증

사용자의 요청에서 다음을 파악한다:
- **주제**: 어떤 레이어/기술/패턴에 관한 규칙인가?
- **범위**: 프로젝트 전역인가, 특정 레이어에 국한되는가?
- **유형**: 코딩 규칙, 테스트 규칙, 아키텍처 규칙, 버그/오류 패턴 중 무엇인가?

### 검증 — 내용 추가 전 반드시 수행

내용을 파악한 뒤, **추가하기 전에** 아래 항목을 점검한다.
문제가 발견되면 **반드시 사용자에게 질문**하고 확인을 받은 뒤 진행한다.

| 검증 항목 | 확인 방법 |
|---------|---------|
| 기존 문서와 모순이 없는가 | 관련 파일을 읽고 기존 규칙과 충돌 여부 확인 |
| 용어·개념이 명확한가 | 약어, 원칙명 등 모호한 표현은 사용자에게 확인 |
| 기존 문서와 중복되지 않는가 | 이미 동일한 규칙이 다른 파일에 기술되어 있는지 확인 |
| 여러 파일에 영향을 미치는가 | 단일 파일 추가로 충분한지, 다른 파일도 업데이트가 필요한지 판단 |

**질문이 필요한 예시:**
- 약어나 원칙명이 불명확할 때 → "OCP를 의미하시나요, 아니면 다른 원칙인가요?"
- 기존 문서와 표면적으로 충돌할 때 → "현재 문서에 X로 명시되어 있는데, 이를 Y로 변경하는 건가요?"
- 범위가 모호할 때 → "이 규칙이 `domain` 모듈에만 적용되나요, 모든 모듈에 적용되나요?"

---

## Step 2: 파일 결정 규칙

### 기존 파일에 추가하는 경우

| 컨벤션 주제 | 파일 |
|------------|------|
| 레이어 구조, 의존성 방향, Facade/Service 사용 기준, Controller 원칙 | `.ai/architecture/architecture.md` |
| DTO 네이밍, record 패턴, 변환 메서드(`toCriteria`, `toCommand`) | `.ai/architecture/dto-rules.md` |
| Read Model(`*Info`), 여러 도메인 조합 방식 | `.ai/architecture/read-model.md` |
| 비즈니스 로직 위치, Cache-DB Fallback, 도메인 분리 | `.ai/architecture/domain-rules.md` |
| ControllerDocs 패턴, Swagger 어노테이션, `@ParameterObject`, `@Hidden` | `.ai/architecture/controller-docs-pattern.md` |
| TDD 사이클, 레이어별 테스트 전략, Reactive 테스트 규칙 | `.ai/architecture/tdd-rules.md` |
| `@DisplayName`, 메서드명 형식, Assertion 스타일 | `.ai/architecture/test-code-rules.md` |
| 프로젝트 전역 규칙, 브레인스토밍 체크리스트, PR 체크리스트, 빌드/실행 명령 | `CLAUDE.md` |
| 기술 부채 항목 | `docs/tech-debt.md` |

### 새 파일을 생성하는 경우

기존 파일 중 주제가 맞는 것이 **없을 때**만 신규 파일을 만든다.

- 파일 위치: `.ai/architecture/{topic}.md`
- 파일명: 주제를 명확히 나타내는 kebab-case
- 생성 후 `CLAUDE.md`의 `## 가이드 문서` 섹션에 `@.ai/architecture/{topic}.md` 참조를 추가해 신규 세션에서도 자동 로드되게 한다.

---

## Step 3: 내용 추가 규칙

### 기존 파일에 추가할 때

1. 파일을 읽고 기존 섹션 구조를 파악한다
2. 가장 관련성 높은 섹션 아래에 추가한다
3. 기존 문서의 어조, 포맷(표/코드블록/리스트)과 일관성을 유지한다
4. ✅/❌ 코드 예시 패턴이 있는 파일이면 동일한 패턴으로 작성한다

### 새 파일을 생성할 때

다음 구조를 기본으로 한다:

```markdown
# {주제} Rules

## 개요
(한 줄 설명)

## 규칙

### {소주제}
(내용)

## 코드 예시

\`\`\`java
// ✅ Good
...

// ❌ Bad
...
\`\`\`
```

---

## Step 4: 결과 보고

추가 완료 후 다음을 알린다:

```
✅ {파일경로} 에 추가했습니다.
추가한 내용: {한 줄 요약}
```

새 파일을 생성했다면:
```
✅ {파일경로} 를 새로 생성하고 CLAUDE.md에 참조를 추가했습니다.
```

---

## 판단이 애매한 경우

- **두 파일 모두 해당될 때**: 더 구체적인 파일에 추가한다 (예: DTO 관련이면 `dto-rules.md` > `architecture.md`)
- **완전히 새로운 기술 영역** (예: jOOQ, Kafka, Redis 패턴): `.ai/architecture/` 신규 파일 생성
- **특정 버그 패턴/트러블슈팅**: `docs/YYYY-MM-DD-{topic}.md` 리포트 형식으로 생성 (CLAUDE.md 참조 추가 불필요)
