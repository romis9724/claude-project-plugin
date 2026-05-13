---
name: project-inbox
description: >
  AI agent 비대기 상태에서 사용자가 원격으로 다음 task를 큐잉할 수 있는 inbox를 표시한다.
  로컬 .claude/inbox.md + GitHub Issues(label:si-project-inbox)를 합쳐 "다음 할 일" 큐 1개 화면으로 출력.
  새 세션 진입·작업 완료 시점에 호출. 30줄 이하, 읽기 전용.
  트리거: "inbox", "받은 일감", "원격 지시", "다음 task 큐", "project inbox"
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(gh issue list*)
  - Bash(git log*)
---

# /si-project:project-inbox — 원격 작업 큐 표시

AI agent 작업 완료 후 또는 새 세션 진입 시, 사용자가 모바일·웹·다른 디바이스에서 큐잉해둔 "다음 할 일"을 한 화면으로 보여준다.

## 설계 의도

Claude Code CLI 세션은 사용자가 콘솔 앞에 있을 때만 지시 가능. 본 skill은 그 갭을 채운다:
- **로컬 채널** (`.claude/inbox.md`): 폐쇄망·오프라인 케이스. git push로 어디서든 추가
- **원격 채널** (GitHub Issues `label:si-project-inbox`): 모바일 GitHub 앱·웹에서 즉시 추가
- **자동 도출** (`/si-project:project-check` 권장 액션): 산출물 모호도·체크리스트 미반영 등에서 자동 도출

읽기 전용. 처리는 사용자가 수동 (체크박스 마킹 / `gh issue close`).

> Anthropic 공식 도구(claude.ai/code 웹, IDE 확장, PushNotification)를 먼저 활용하고, 그 갭을 본 skill이 메운다. CLAUDE.md "원격 작업 큐" 섹션 참조.

---

## STEP 0 — 사전 확인

- `CLAUDE.md` 존재 여부 → 없으면 "/si-project:project-setup이 먼저 필요합니다" 안내 후 중단
- 출력은 30줄 이하로 압축

---

## STEP 1 — 로컬 inbox 스캔 (`.claude/inbox.md`)

파일 존재 시 Read. 형식:

```markdown
# Project Inbox

## P0 (긴급)
- [ ] 결제 PG 연동 명세 받기

## P1 (중요)
- [ ] /admin 화면 와이어프레임 검토
- [x] 완료된 항목 (처리 후 사용자가 [x] 마킹)

## P2 (보통)
- [ ] 매뉴얼 한글 검수
```

추출 규칙:
- `- [ ]`로 시작하는 줄만 → 미처리 항목
- `## P0` / `## P1` / `## P2` 헤더로 우선순위 분류
- `[x]` 항목은 출력 제외 (사용자가 직접 정리)

파일 없으면 본 섹션 출력 자체를 생략 (가이드 1줄만).

---

## STEP 2 — GitHub Issues 스캔 (label 기반)

`gh` CLI 사용 가능 여부 확인:

```
Bash: gh issue list --label si-project-inbox --state open --limit 20 --json number,title,labels,assignees,url
```

- **성공**: JSON 파싱해 표시 (번호·제목·담당자·URL)
- **실패** (gh 미설치 / 미인증 / 네트워크 차단 / 폐쇄망): 본 섹션 출력 생략, 메시지 없이 자연스럽게 스킵
- 라벨이 0건이면 "(없음)" 표시

> CLAUDE.md에 `INBOX_LABEL` 변수가 정의되어 있으면 그 값 우선 사용 (기본: `si-project-inbox`).

---

## STEP 3 — 자동 도출 권장 (project-check 기반)

다음 4축에서 미해결 항목을 1-3개로 압축:
1. `docs/01-requirements/`의 `명확도: 모호/미정` 행 수 → "{N}건 명확화 필요 (`/si-project:project-check`)"
2. CLAUDE.md `## 미설정 항목`의 `- [ ]` 미체크 줄 수
3. 산출물 내 `> ⚠️ TODO` 카운트 (최근 변경 산출물 기준 — `git log --name-only -5`)
4. `.env.example`와 `.env` 차이의 빈 슬롯 수

각 1줄로 압축. 0건이면 생략.

> 본 섹션은 `/si-project:project-check`와 중복되지 않게 **카운트 + 한 줄 권장**만 표시. 상세는 project-check 호출 안내.

---

## STEP 4 — 통합 출력 (30줄 이하)

다음 형식으로 출력:

```
📥 Inbox — 다음 할 일 큐

🗂 로컬 (.claude/inbox.md):
- [P0] 결제 PG 연동 명세 받기
- [P1] /admin 화면 와이어프레임 검토
- [P2] 매뉴얼 한글 검수

🔗 GitHub Issues (label: si-project-inbox):
- #42 [P0] 인증서 갱신 절차 정의 — @user (https://github.com/.../issues/42)
- #45 [P2] 매뉴얼 한글 검수 — @teammate

🎯 자동 권장 (project-check 4축):
- requirements 모호 항목 3건 → `/si-project:project-check`로 보강
- CLAUDE.md ## 미설정 항목 2건 미체크
- .env 빈 슬롯 5개

💡 처리 방법:
- 로컬: inbox.md에서 [ ] → [x] 마킹
- GitHub: `gh issue close <N>` 또는 모바일 앱에서 close
- 입력 채널 안내: CLAUDE.md "원격 작업 큐" 섹션 참조
```

빈 섹션은 통째 생략. 모두 0건이면 `📭 큐가 비어 있습니다.` 1줄만 출력.

---

## STEP 5 — 후속 액션 제안 (선택)

inbox 우선순위 최상위 항목이 있으면, 마지막에 1줄 제안:

```
다음으로 추천: "{최상위 P0 항목}" — 진행하시겠습니까?
```

P0가 없으면 P1 → P2 순. 본 줄은 30줄 한도 안에 포함.

---

## 사용 예시

```
사용자: /si-project:project-inbox
Claude:
  → .claude/inbox.md 읽기 (3건)
  → gh issue list --label si-project-inbox (2건)
  → project-check 4축 자동 카운트 (3건)
  → 통합 30줄 출력
  → "다음 추천: 결제 PG 연동 명세 받기"
```

```
사용자(폐쇄망): /si-project:project-inbox
Claude:
  → .claude/inbox.md 읽기
  → gh 호출 실패 → GitHub 섹션 자연 스킵
  → 로컬 + 자동 권장만 출력
```

```
사용자: 작업 끝났어. 다음 뭐 할까?
Claude (자동 추천):
  → 트리거 매칭 "다음 뭐"
  → /si-project:project-inbox 호출 권장
```
