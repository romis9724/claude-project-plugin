---
title: 형상 관리 절차서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{HARNESS_ENGINEER}}"
---

# 형상 관리 절차서: {{PROJECT_NAME_KR}}

## 브랜치 전략

**전략**: {{BRANCH_STRATEGY}} (Git Flow / Trunk-Based / GitHub Flow)

```
main          ← 운영 배포 (보호됨, 직접 push 금지)
  └─ develop  ← 통합 브랜치
       ├─ feature/{{TICKET_ID}}-{{FEATURE_NAME}}  ← 기능 개발
       ├─ fix/{{TICKET_ID}}-{{BUG_NAME}}          ← 버그 수정
       └─ hotfix/{{TICKET_ID}}-{{ISSUE_NAME}}     ← 긴급 수정
```

- `main` 브랜치: PR + 승인 2인 + CI 통과 후 병합만 허용
- `develop` 브랜치: PR + 승인 1인 + CI 통과 후 병합

## 커밋 규칙

**포맷**: `type(scope): 한글 메시지`

| type | 용도 |
|------|------|
| feat | 새 기능 |
| fix | 버그 수정 |
| docs | 문서 수정 |
| refactor | 리팩토링 |
| test | 테스트 추가/수정 |
| chore | 빌드/설정 변경 |
| perf | 성능 개선 |
| security | 보안 패치 |

**예시**: `feat(auth): JWT 토큰 갱신 로직 구현`

**금지 사항**:
- 의미 없는 메시지 (fix, update, wip 단독 사용)
- `.env` 파일 커밋
- 빌드 산출물 커밋 (`target/`, `dist/`, `__pycache__/`)

## 버전 관리 (SemVer)

`MAJOR.MINOR.PATCH` 규칙:
- MAJOR: 하위 호환 불가 변경
- MINOR: 하위 호환 기능 추가
- PATCH: 버그 수정

## CI/CD 파이프라인

| 단계 | 트리거 | 작업 | 통과 기준 |
|------|--------|------|---------|
| CI (PR) | PR 생성/업데이트 | 빌드 + 테스트 + 린터 + 보안 스캔 | 모두 통과 |
| CI (develop) | develop 병합 | 빌드 + 통합 테스트 + 스테이징 배포 | 모두 통과 |
| CD (main) | main 병합 | 운영 배포 + 연기 테스트 | 승인 후 진행 |

**하네스 도구**: {{CI_TOOL}} (GitHub Actions / GitLab CI / Jenkins)

## 환경별 설정 관리

| 환경 | 설정 방식 | 비밀값 관리 |
|------|----------|------------|
| dev | `.env.local` (git 제외) | 개발자 로컬 |
| staging | CI/CD 환경변수 | {{SECRET_MANAGER}} |
| prod | {{SECRET_MANAGER}} | {{SECRET_MANAGER}} |

> **규칙**: API 키, DB 비밀번호, 인증 토큰은 절대 코드/설정 파일에 하드코딩 금지

## 관련 문서

- [품질 관리 계획서](quality-plan.md)
- [시스템 아키텍처 정의서](../02-architecture/system-architecture.md)
