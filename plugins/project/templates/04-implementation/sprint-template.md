---
title: "스프린트 {{N}} 계획"
project: "{{PROJECT_NAME_KR}}"
sprint: "{{N}}"
period: "{{START_DATE}} ~ {{END_DATE}}"
status: 계획
---

# Sprint {{N}} 계획: {{PROJECT_NAME_KR}}

> **기간**: {{START_DATE}} ~ {{END_DATE}} ({{SPRINT_LENGTH}})
> **스프린트 목표**: {{SPRINT_GOAL}}

## 스프린트 목표

{{SPRINT_GOAL_DETAIL}}

## 선택된 백로그 항목

| ID | 에픽 | 유저 스토리 | 담당 | 포인트 | 상태 |
|----|------|-----------|------|--------|------|
| US-{{ID_1}} | {{EPIC_1}} | {{STORY_1}} | {{DEV_1}} | {{PT_1}} | 시작전 |
| US-{{ID_2}} | {{EPIC_2}} | {{STORY_2}} | {{DEV_2}} | {{PT_2}} | 시작전 |

**총 스토리 포인트**: {{TOTAL_POINTS}}

## 스프린트 리스크

- {{SPRINT_RISK_1}}
- {{SPRINT_RISK_2}}

## 완료 기준 (DoD)

- [ ] 코드 리뷰 완료 (리뷰어 2인 이상 승인)
- [ ] 단위 테스트 통과 (커버리지 ≥ {{COVERAGE_TARGET}}%)
- [ ] 통합 테스트 통과
- [ ] 린터/포매터 통과 (CI 파이프라인)
- [ ] 보안 취약점 스캔 통과
- [ ] 관련 문서 업데이트
- [ ] 스테이징 환경 배포 및 검증

## 스프린트 결과 (종료 후 작성)

| 항목 | 계획 | 실적 |
|------|------|------|
| 완료 스토리 | {{PLANNED_STORIES}} | |
| 완료 포인트 | {{TOTAL_POINTS}} | |
| 버그 발견 | - | |
| 기술 부채 | - | |
