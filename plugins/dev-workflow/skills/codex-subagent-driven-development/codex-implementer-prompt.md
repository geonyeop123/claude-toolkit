# Codex Implementer Prompt Template

아래를 복사해서 `[...]` 부분을 채워 Codex에 전달한다.

---

```
## 작업 컨텍스트

[프로젝트 개요 1-2줄: 예) Spring WebFlux + R2DBC 멀티모듈 프로젝트]

이 태스크는 구현 플랜의 [N번째] 태스크입니다.
플랜 위치: [docs/superpowers/plans/YYYY-MM-DD-feature.md]

## 태스크 내용

[플랜에서 해당 태스크 전체 텍스트를 그대로 붙여넣기]

## 아키텍처 규칙 (반드시 준수)

- 모든 테스트는 TDD 순서로 작성: 실패하는 테스트 먼저 → 실패 확인 → 최소 구현 → 통과 확인
- `./gradlew` 명령으로 실제 테스트를 실행하여 FAIL/PASS를 반드시 확인한다
- 테스트 통과 후 반드시 git commit을 수행한다
- `.block()` 사용 금지, HTTP 응답 코드는 항상 200

## 시작 전 확인

아래 사항이 불명확하면 작업을 시작하기 전에 질문한다:
- 의존 파일/클래스가 이미 존재하는지 확실하지 않은 경우
- 구현 방식이 여러 가지로 해석될 수 있는 경우
- 플랜의 코드가 실제 코드베이스와 다른 경우

## 완료 보고 형식 (반드시 마지막에 출력)

---
STATUS: [DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED]

구현 내용:
- [변경/생성 파일 목록]

테스트 결과:
- 실행 명령: [./gradlew ...]
- 결과: [BUILD SUCCESSFUL / FAILED]

자체 검토:
- [플랜 요구사항과 구현 내용 비교]
- [추가하거나 제거한 것이 있으면 명시]

우려사항 (DONE_WITH_CONCERNS인 경우):
- [우려 내용]

블로커 (BLOCKED/NEEDS_CONTEXT인 경우):
- [필요한 정보 또는 차단 원인]
---
```

---

## STATUS 정의

| STATUS | 의미 |
|--------|------|
| `DONE` | 모든 요구사항 구현, 테스트 통과, 커밋 완료 |
| `DONE_WITH_CONCERNS` | 완료했지만 플래그할 우려사항이 있음 |
| `NEEDS_CONTEXT` | 작업을 계속하려면 추가 정보 필요 |
| `BLOCKED` | 진행 불가 — 아키텍처 판단, 예상 외 코드 상태 등 |

## 파싱 방법

Codex 출력에서 STATUS 줄 추출:
```bash
grep "^STATUS:" output.txt
```
