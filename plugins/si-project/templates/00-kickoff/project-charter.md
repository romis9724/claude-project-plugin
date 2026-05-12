---
title: 프로젝트 정의서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{PM}}"
---

# 프로젝트 정의서: {{PROJECT_NAME_KR}}

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| 프로젝트명 | {{PROJECT_NAME_KR}} ({{PROJECT_NAME_EN}}) |
| 목적 | {{PURPOSE}} |
| 기간 | {{START_DATE}} ~ {{END_DATE}} |
| 예산 | {{BUDGET}} |
| PM | {{PM}} |

## 목표

- {{GOAL_1}}
- {{GOAL_2}}

## 범위

### 포함 범위 (In-Scope)
- {{IN_SCOPE_1}}

### 제외 범위 (Out-of-Scope)
- {{OUT_SCOPE_1}}

## 이해관계자

| 역할 | 담당자 | 책임 |
|------|--------|------|
| PMO | {{PMO}} | 프로젝트 거버넌스 |
| PM | {{PM}} | 일정·범위·리스크 관리 |
| 아키텍트 | {{ARCHITECT}} | 기술 아키텍처 결정 |
| DBA | {{DBA}} | 데이터베이스 설계·관리 |
| 하네스 엔지니어 | {{HARNESS}} | CI/CD 환경 구성 |
| 보안 전문가 | {{SECURITY}} | 보안 요구사항 정의·검토 |

## 주요 제약사항

- **기술**: {{TECH_CONSTRAINT}}
- **일정**: {{SCHEDULE_CONSTRAINT}}
- **예산**: {{BUDGET_CONSTRAINT}}
- **법규/규제**: {{REGULATION}}

## 성공 기준

| 지표 | 목표값 | 측정 방법 |
|------|--------|-----------|
| {{KPI_1}} | {{TARGET_1}} | {{MEASURE_1}} |

## 마일스톤

| 단계 | 주요 산출물 | 예정일 |
|------|-----------|--------|
| 착수 | 프로젝트 정의서, 수행계획서 | {{DATE_M1}} |
| 요구분석 | 요구사항 기술서 | {{DATE_M2}} |
| 아키텍처 | SW 아키텍처 기술서, 보안 정의서 | {{DATE_M3}} |
| 설계 | DB 설계서, API 설계서 | {{DATE_M4}} |
| 구현 완료 | 제품 기능 구현 | {{DATE_M5}} |
| 인도 | 인수 테스트 완료 | {{DATE_M6}} |
