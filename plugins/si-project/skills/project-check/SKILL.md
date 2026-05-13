---
name: project-check
description: >
  SI 프로젝트의 "구현 준비도(명확성 게이트)"를 점검한다.
  요구사항 산출물의 명확도 등급(명확/모호/미정), TBD 패턴, `.env` 빈 슬롯,
  CLAUDE.md `## 미설정 항목`, `.claude/project-context.json` 누락 필드를 스캔해
  0~100점 준비도 점수와 다음 권장 액션 3건을 출력한다.
  읽기 전용 — 파일 수정 없음. 30줄 이하 출력.
  본격 구현 시작 전 또는 마일스톤 진입 직전 호출.
  트리거: "구현 시작해도 되나", "준비도 점검", "명확도 체크", "모호한 부분 확인", "project check"
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(ls *)
  - Bash(Get-ChildItem *)
---

# /si-project:project-check — 구현 준비도(명확성) 게이트 점검

SI 프로젝트에서 **모호한 요구사항·미설정 외부 정보 상태로 구현이 진행되는 사고**를 막기 위한 자기 점검 도구.

설계 원칙:
- **읽기 전용** — CLAUDE.md·산출물·project-context.json 일절 수정 X
- 30줄 이하 출력 (한눈에 스캔)
- 추측 금지. 파일에 있는 사실만 카운트
- 점수 산정은 단순·재현 가능한 공식 (블랙박스 금지)

---

## STEP 0 — 사전 확인 + lessons 참고

1. `CLAUDE.md` 존재 여부 → 없으면 "이 디렉토리는 `/si-project:project-setup`으로 초기화되지 않았습니다" 안내 후 중단
2. `.claude/lessons.md` 존재 시 Read → 1줄 압축해 출력 말미에 "참고 lesson N건"으로 노출 (없으면 스킵)

---

## STEP 1 — 4축 스캔

존재하지 않는 항목은 0으로 카운트. 에러로 중단 X.

### 축 1. 요구사항 명확도 (`docs/01-requirements/**/*.md` 전체)

Grep 카운트 (output_mode: count 또는 files_with_matches → 총합):

| 카운트 | 패턴 |
|------|------|
| `CLARITY_CLEAR` | `명확도\s*[:|│]\s*명확` |
| `CLARITY_VAGUE` | `명확도\s*[:|│]\s*모호` |
| `CLARITY_UNDECIDED` | `명확도\s*[:|│]\s*미정` |
| `TBD_COUNT` | `\{\{TBD\}\}\|^\s*TBD\s*$\|\bTBD\b` (테이블 셀 포함) |

### 축 2. 외부 정보 미설정 (`.env.example` 또는 `.env`)

루트의 `.env.example` 우선, 없으면 `.env`.

- `ENV_TOTAL_KEYS` = `^[A-Z][A-Z0-9_]*=` 매칭 줄 수
- `ENV_EMPTY_COUNT` = `^[A-Z][A-Z0-9_]*=\s*(#.*)?$` 매칭 줄 수 (값 없는 키 — `#` 주석만 있는 경우 포함)

파일 없으면 둘 다 0.

### 축 3. CLAUDE.md `## 미설정 항목` 섹션

CLAUDE.md에서 `## 미설정 항목` ~ 다음 `## ` 사이를 Read offset/limit으로 추출:

- `UNSET_COUNT` = `^- \[ \]` 줄 수 (미체크)
- `PASSED_COUNT` = `^- \[x\]` 줄 수 (체크 완료)

섹션 없으면 둘 다 0 (단, 본 섹션은 v2.1.0 setup이 기본 생성하므로 누락 시 안내 1줄 추가).

### 축 4. 행정 정보 (`.claude/project-context.json`)

`CONTEXT_MISSING` = 3에서 시작, 다음 키 발견 시 -1:
- `client` (또는 `client_org`, `client.name`)
- `contract.acceptance` (또는 `acceptance`, `acceptance_criteria`)
- `schedule.delivery_date` (또는 `delivery_date`, `due_date`)

파일 없으면 `CONTEXT_MISSING = 3`.

---

## STEP 2 — 준비도 점수 산정

```
clarity_total = CLARITY_CLEAR + CLARITY_VAGUE + CLARITY_UNDECIDED
clarity_ratio = CLARITY_CLEAR / max(clarity_total, 1)        # 명확 비율
env_ratio     = 1 - (ENV_EMPTY_COUNT / max(ENV_TOTAL_KEYS, 1))  # .env 없으면 1.0
unset_total   = PASSED_COUNT + UNSET_COUNT
unset_ratio   = PASSED_COUNT / max(unset_total, 1)             # 섹션 없으면 1.0
context_ratio = 1 - (CONTEXT_MISSING / 3)

READINESS_SCORE = round(
    clarity_ratio * 50 +
    env_ratio     * 20 +
    unset_ratio   * 15 +
    context_ratio * 15
)
```

페널티 (점수에서 차감):
- `TBD_COUNT > 20` → -10
- `CLARITY_UNDECIDED > 5` → -10

상한 100, 하한 0. clarity_total == 0 (요구사항 산출물이 아예 비어 있음)이면 점수 무관하게 등급 **"구현 시작 비권장"** 강제.

등급:
- ≥80: **통과** (구현 진행 가능)
- 50~79: **보강 필요** (모호 항목 명확화 후 진행)
- <50: **구현 시작 비권장** (핵심 요구사항·외부 정보 확정 후 재점검)

---

## STEP 3 — 출력 (30줄 이하)

```
🔍 구현 준비도 점검

📊 종합 점수: {READINESS_SCORE}/100  ({등급})

🎯 요구사항 명확도 (docs/01-requirements/)
  - 명확: {CLARITY_CLEAR}  |  모호: {CLARITY_VAGUE}  |  미정: {CLARITY_UNDECIDED}
  - TBD/미작성: {TBD_COUNT}건

🔐 외부 정보·미설정
  - .env 빈 슬롯: {ENV_EMPTY_COUNT}/{ENV_TOTAL_KEYS}
  - CLAUDE.md ## 미설정 항목: 채워짐 {PASSED_COUNT} / 남음 {UNSET_COUNT}
  - project-context.json 누락 필드: {CONTEXT_MISSING}/3

🎬 다음 권장 액션 (최대 3건)
  1. {모호·미정 요구사항 중 우선순위 상위 1건 — 파일·ID 명시}
  2. {.env 빈 슬롯 첫 키}
  3. {## 미설정 항목 첫 미체크 항목 또는 project-context.json 누락 필드}

ℹ️ 참고 lesson: {.claude/lessons.md N건 / 가장 최근 1건 제목}
```

권장 액션이 0건(통과 상태)이면 "(없음 — 구현 진행 가능)" 1줄로 압축.

**길이 제한**: 본 출력은 30줄 이하. 카운트 정보를 줄여서라도 권장 액션 3건은 유지.

---

## STEP 4 — 종료

본 스킬은 쓰기 작업 없음.
- CHANGELOG.md 업데이트 X
- project-context.json 업데이트 X
- CLAUDE.md `## 미설정 항목` 자동 수정 X (사용자가 직접 채워 줄을 지우거나 `[x]`로 변경)

후속 액션 제안만 출력하고 종료.

---

## 사용 예시

```
사용자: /si-project:project-check
Claude:
  → CLAUDE.md, docs/01-requirements/, .env, project-context.json 스캔
  → 점수: 62/100 (보강 필요)
  → "FR-007 모호 → 사용자 권한 정책 확정 필요" 등 권장 액션 출력
```

```
사용자: 이제 구현 시작해도 될까?
Claude:
  → /si-project:project-check 트리거
  → 30줄 요약 출력 후 통과/보강/비권장 한 줄로 결론
```

---

## 비판적 자가 점검 (이 스킬 설계자 메모)

- 본 점수는 **신호**일 뿐 **차단**이 아니다. AI가 매번 부르도록 강제하지 못함 → CLAUDE.md "AI 에이전트 작업 원칙 0번"에서 호출 권장으로 보완
- `명확도: 명확`을 작성자가 잘못 표기하면 점수는 위양성으로 높아질 수 있음 → 작성자 책임. 본 스킬은 "기록된 사실"만 본다
- 페널티 임계치(TBD>20, 미정>5)는 휴리스틱. 프로젝트 규모에 따라 조정 필요 시 사용자가 본 SKILL.md 직접 수정
