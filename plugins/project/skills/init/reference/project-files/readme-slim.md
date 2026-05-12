# {KR_NAME}

> {PURPOSE}

**컴포넌트 구성** ({MONOREPO_LAYOUT}) | **기간** {PERIOD} | **인프라** {INFRA}

## 빠른 시작

각 컴포넌트는 자체 framework CLI로 초기화 (mono-repo):
- Backend: `cd backend && {STACK_BACKEND 표준 초기화 명령}`
- User FE: `cd user-fe && {STACK_USER_FE 표준 초기화 명령}`
- (사용 컴포넌트만 출력)

**Commit 가드 활성화** (1회): `pip install pre-commit && pre-commit install`

## 핵심 문서

- **AI 에이전트 컨텍스트**: [CLAUDE.md](CLAUDE.md) — 스택·컨벤션·보안·Git 규약 전체
- **요구사항**: [docs/01-requirements/requirements.md](docs/01-requirements/requirements.md)
- **아키텍처**: [docs/02-architecture/software-architecture.md](docs/02-architecture/software-architecture.md)
- **보안**: [docs/02-architecture/security-definition.md](docs/02-architecture/security-definition.md)
- **변경 이력**: [CHANGELOG.md](CHANGELOG.md)
- **Git 원격 셋업** (1회): [scripts/git-setup.md](scripts/git-setup.md)

> GitHub 외 SCM(GitLab/Bitbucket) 사용 시 `.github/` 파일을 각 SCM 등가 위치로 옮길 것 (GitLab: `.gitlab/merge_request_templates/Default.md`).
