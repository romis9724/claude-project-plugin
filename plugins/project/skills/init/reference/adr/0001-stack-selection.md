---
title: "ADR-0001: 초기 기술 스택 선택"
status: Accepted
date: {DATE}
deciders: [{OWNER 또는 TBD}]
tags: [stack, backend, frontend, mobile, db, infra]
---

# ADR-0001: 초기 기술 스택 선택

## Status

**Accepted** ({DATE})

> 본 ADR은 init이 자동 생성. 팀 협의로 변경 시 신규 ADR로 Supersede.

## Context

본 프로젝트는 다음 시스템 유형으로 정의됨: {SYSTEM_TYPES}.

주요 제약:
- **데이터 민감도**: {DATA_SENSITIVITY}
- **적용 규제**: {COMPLIANCE}
- **인프라·망환경**: {INFRA}
- **기간**: {PERIOD}
- **CI/CD**: {HARNESS}

이러한 제약 하에서 각 컴포넌트의 기술 스택을 선택해야 한다.

## Decision

다음 스택을 채택한다:

| 컴포넌트 | 선택 | 비고 |
|---------|------|------|
| Backend | {STACK_BACKEND} | {STACK_BACKEND_USE} |
| 사용자 Frontend | {STACK_USER_FE} | {STACK_USER_FE_USE} |
| 관리자 Frontend | {STACK_ADMIN_FE} | 모드: {STACK_ADMIN_FE_MODE} |
| Mobile | {STACK_MOBILE} | 유형: {STACK_MOBILE_TYPE} |
| RDB | {DB_RDB} | |
| NoSQL | {DB_NOSQL} | |
| 검색 | {DB_SEARCH} | |
| 분석 | {DB_ANALYTICS} | |
| 인증 | {AUTH} | |

## Consequences

**긍정**:
- 컴포넌트별 표준 스택으로 채용·교육·외부 협업이 용이
- 위 제약(데이터 민감도·규제·인프라)에 부합

**부정**:
- 특정 스택 종속 (예: framework upgrade 시 영향)
- 신규 기술 도입 시 본 ADR을 Supersede하는 신규 ADR 필요

**잔존 위험**:
- 운영 중 성능·라이선스·보안 이슈 발견 시 재검토
- 의존성 버전은 본 ADR 외 SCA 도구·docs/02-architecture/software-architecture.md에서 별도 관리

## Alternatives Considered

본 ADR은 init 시점 자동 생성이라 대안 검토 기록은 비어있다. 사용자가 사후 추가 가능.

| 대안 | 기각 사유 |
|------|---------|
| (TBD — 검토했다면 기록) | |

## References

- 관련 문서:
  - [software-architecture.md](../software-architecture.md)
  - [security-definition.md](../security-definition.md)
  - CLAUDE.md (스택 요약)
- 외부 자료: 없음
