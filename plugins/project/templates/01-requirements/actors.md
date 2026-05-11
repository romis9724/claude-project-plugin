---
title: 사용자(행위자) 정의서
project: "{{PROJECT_NAME_KR}}"
version: 0.1
status: 초안
date: "{{DATE}}"
owner: "{{PM}}"
---

# 사용자(행위자) 정의서: {{PROJECT_NAME_KR}}

## 행위자 목록

| ID | 행위자명 | 유형 | 설명 | 권한 수준 |
|----|---------|------|------|-----------|
| A01 | {{ACTOR_1}} | 사람 | {{ACTOR_DESC_1}} | {{AUTH_LEVEL_1}} |
| A02 | {{ACTOR_2}} | 사람 | {{ACTOR_DESC_2}} | {{AUTH_LEVEL_2}} |
| A03 | {{ACTOR_3}} | 시스템 | {{ACTOR_DESC_3}} | API |

## 주요 사용자 페르소나

### {{PERSONA_1_NAME}}
- **역할**: {{PERSONA_1_ROLE}}
- **기술 수준**: {{PERSONA_1_SKILL}}
- **목표**: {{PERSONA_1_GOAL}}
- **불편사항**: {{PERSONA_1_PAIN}}

### {{PERSONA_2_NAME}}
- **역할**: {{PERSONA_2_ROLE}}
- **기술 수준**: {{PERSONA_2_SKILL}}
- **목표**: {{PERSONA_2_GOAL}}
- **불편사항**: {{PERSONA_2_PAIN}}

## 외부 시스템 행위자

| 시스템명 | 연동 방식 | 역할 | 담당 팀 |
|---------|----------|------|---------|
| {{EXT_SYS_1}} | {{PROTOCOL_1}} | {{ROLE_1}} | {{TEAM_1}} |
| {{EXT_SYS_2}} | {{PROTOCOL_2}} | {{ROLE_2}} | {{TEAM_2}} |

## 관련 문서

- [요구사항 기술서](requirements.md)
- [보안 정의서](../02-architecture/security-definition.md)
