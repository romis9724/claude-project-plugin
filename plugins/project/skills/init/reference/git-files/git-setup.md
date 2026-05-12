# Git 원격 정책 셋업 가이드

본 문서는 init이 자동 적용할 수 없는 원격(GitHub/GitLab) 정책을 사람이 1회 실행하기 위한 체크리스트.

## 1. Remote 등록

```bash
git remote add origin <REPO_URL>
git push -u origin main
```

## 2. Branch protection (main, develop이 있으면 develop도)

GitHub: Settings → Branches → Add branch protection rule
- Branch name pattern: `main` (그리고 `develop`)
- ✅ Require a pull request before merging
- ✅ Require approvals: {CODE_REVIEW_POLICY 기반 — 1 또는 2}
- ✅ Dismiss stale pull request approvals
- ✅ Require status checks to pass before merging
  - 필수 CI workflow 추가 (예: `ci / lint`, `ci / test`, `ci / sast`)
- ✅ Require conversation resolution
- ✅ Restrict force push
- ✅ Restrict deletions
- (선택) Require linear history

## 3. CODEOWNERS 활성

`.github/CODEOWNERS`의 TODO 부분을 팀원·팀 ID로 교체. branch protection에서 ✅ Require review from Code Owners 체크.

## 4. Required status checks

Q5에서 선택한 도구들의 CI job 이름:
- 린터: {LINTER_*}
- 테스트: {TEST_UNIT_*}, {TEST_E2E}
- SAST: {STATIC_SAST}
- SCA: {STATIC_SCA}
- 이미지: {STATIC_IMAGE}

## 5. Issue / PR 라벨

- type: feat / fix / docs / refactor / test / chore / security / perf / build / ci
- scope: backend / user-fe / admin-fe / mobile / infra / docs / meta
- priority: P0 / P1 / P2

## 6. (Mobile 있을 때) Git LFS

```bash
git lfs install
git lfs track "*.keystore" "*.aab" "*.ipa"
git add .gitattributes
git commit -m "chore(meta): enable Git LFS for mobile binaries"
```
