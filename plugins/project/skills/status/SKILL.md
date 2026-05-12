---
name: status
description: >
  현재 프로젝트의 진행 현황과 오늘·이번 주 권장 액션을 30줄 이하로 요약한다.
  세션 재진입·일감 파악·납기 추적 시 사용. 쓰기 작업 없음(읽기 전용).
  트리거: "현황", "오늘 할 일", "이번 주", "프로젝트 상태", "어디까지 했지", "이어서", "session resume", "status"
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git log*)
  - Bash(git status*)
  - Bash(git branch*)
  - Bash(ls *)
  - Bash(Get-ChildItem *)
---

# /project:status — 프로젝트 현황·오늘 할 일 요약

새 세션 진입·일감 파악 시점에 현재 프로젝트의 상태를 빠르게 파악한다. **읽기 전용** — 파일을 생성·수정하지 않는다.

설계 원칙:
- 30줄 이하 출력 (사용자 한눈에 스캔 가능)
- JIRA/Linear 대체 아님. 풀 프로젝트 관리 도구로 확장 금지
- 추측 금지. 파일·git에 있는 사실만 보고

---

## STEP 0 — 사전 확인 + lessons 참고

1. `CLAUDE.md` 존재 여부 → 없으면 "이 디렉토리는 /project:init으로 초기화되지 않았습니다" 안내 후 중단
2. `.claude/lessons.md` 존재 시 Read → 1줄 압축해 출력 말미 "참고 lesson N건"으로 노출 (없으면 스킵)

---

## STEP 1 — 정보 소스 수집 (lazy, 부분 읽기)

다음 소스를 순차 확인. **존재하지 않는 항목은 조용히 건너뛴다** (에러 X).

| 소스 | 추출 방법 | 추출 정보 |
|------|---------|----------|
| `CLAUDE.md` | Read offset=1, limit=20 | 프로젝트명·목적·기간·핵심 스택 |
| `.claude/project-context.json` | Read 전체 (보통 1~3KB) | 현재 마일스톤·납기·이해관계자 |
| `CHANGELOG.md` | Read offset=1, limit=40 | 최근 변경 5~10건 |
| `git log` | `Bash: git log -10 --oneline --decorate` | 최근 커밋 |
| `git status` | `Bash: git status --short --branch` | 현재 브랜치·미커밋 변경 |
| `docs/` 디렉토리 | Glob `docs/**/*.md` | 완성된 산출물 목록 (마일스톤별 카운트) |

git이 초기화되지 않은 경우 git 관련 단계는 스킵 (`fatal: not a git repository` 무시).

---

## STEP 2 — 현재 마일스톤 추정

다음 로직으로 현재 진행 중인 마일스톤을 추정:

1. `.claude/project-context.json`의 `schedule.milestones`에 명시되어 있으면 그것 사용
2. 없으면 `docs/{milestone}/` 디렉토리 중 파일이 가장 많은 곳을 "최근 활동 마일스톤"으로 추정
3. 둘 다 없으면 `00-kickoff` (기본값)

---

## STEP 3 — 5섹션 요약 출력

다음 형식으로 출력. 각 섹션 5줄 이내. 빈 섹션은 "(해당 없음)" 1줄.

```
📋 {프로젝트 한글명} ({EN_NAME})
   목적: {1줄 요약, CLAUDE.md PURPOSE에서}
   기간: {PERIOD} | 현재 진행률 추정: {계산값 또는 "추정 불가"}

🎯 현재 마일스톤: {milestone-id} ({한글명})
   납기: {project-context.schedule.delivery_date 또는 "미정"}
   완료 산출물: docs/{milestone}/에서 N개

🔄 최근 변경 (최근 5건)
   - {git log 한 줄}
   - ...

✅ 완료된 산출물 (마일스톤별)
   - 00-kickoff: N건  |  01-requirements: M건  |  02-architecture: K건
   - ...

🎬 다음 권장 액션 (3건 이내)
   1. {현재 마일스톤의 필수 산출물 중 docs/에 없는 첫 번째}
   2. {git status에 미커밋 변경 있으면 "변경사항 커밋"}
   3. {project-context.json 누락 필드 있으면 "행정 정보 보강"}

ℹ️ 참고 lesson: .claude/lessons.md N건 ({날짜 가장 최근 1개 제목})
```

**길이 제한**: 본 출력은 30줄 이하. 산출물 카운트는 마일스톤 8개를 2줄로 압축. 5번째 권장 액션은 만들지 않음.

---

## STEP 4 — 종료

본 스킬은 쓰기 작업이 없다.
- CHANGELOG.md 업데이트 X (status는 변경 없음)
- project-context.json 업데이트 X
- 후속 액션 제안만 출력하고 종료

---

## 사용 예시

```
사용자: /project:status
Claude:
  → CLAUDE.md, project-context.json, CHANGELOG.md(부분), git log 수집
  → 현재 마일스톤: 01-requirements (docs/01-requirements/ 4건 발견)
  → 30줄 요약 출력
```

```
사용자: 오늘 할 일?
Claude:
  → /project:status 트리거
  → "다음 권장 액션" 섹션을 강조해 출력
```
