# Subagent 활용 패턴 — 구현 단계 가이드

산출물 작성이 끝나고 코드 구현 단계에 진입했을 때, Claude Code의 Agent tool로 다음 패턴을 활용 가능. **자율 사용 — 강제 아님.**

> 본 가이드는 코드 구현 단계용. 산출물 작성·검토 단계의 subagent는 `/si-project:project-document` · `/si-project:project-milestone`이 자동 호출(검토자 패턴, v2.2). 본 문서는 그 후속 단계에 활용.

---

## ✅ 권장 (cross-reference 위험 낮음)

- **단위 테스트 생성**: 구현 코드 입력 → 테스트 작성 subagent
- **코드 리뷰어**: 보안 / 성능 / 스타일 리뷰 (병렬 3개 호출 가능)
- **코드베이스 탐색**: 의존성·호출처 추적 (격리 컨텍스트, 결과 요약만 부모로)
- **문서/주석 생성**: docstring, README 보강

## ⚠️ 신중 (인터페이스 합의 선행)

- **다중 모듈 동시 구현**: 공통 DTO·인터페이스·에러 코드를 부모가 먼저 확정해 사전 패키지로 전달한 뒤에만 병렬

## ❌ 위험 (subagent 분리 금지)

- DB 스키마 변경·마이그레이션
- 인증/인가 로직
- 트랜잭션 경계 코드
- 공통 미들웨어·인터셉터

## 모델 선택 가이드

- `haiku` / `sonnet`: 단순 CRUD·보일러플레이트·테스트·문서
- `opus`: 핵심 알고리즘·보안 로직·아키텍처 결정 코드

---

## 호출 형식 (Claude Code Agent tool 기준)

```
Agent(
  description: "한 줄 작업 요약",
  subagent_type: "general-purpose",   # 또는 전용 agent
  model: "sonnet",                    # 작업 성격에 맞게
  prompt: """
    [컨텍스트] 부모가 정리한 코드·인터페이스·제약
    [작업] 구체 지시 (예: 'src/api/order.py의 createOrder에 대한 단위 테스트 작성')
    [응답 형식] 다른 부연 설명 없이 코드만, 또는 요약 N줄
  """
)
```

**부모 컨텍스트 보호**: subagent 응답 본문 외 사고·근거는 부모로 가져오지 말 것. 격리 이득 보존이 핵심.
