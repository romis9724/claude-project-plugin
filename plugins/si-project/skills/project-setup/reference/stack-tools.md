# Stack → Bash 권한 매핑

`/si-project:project-setup` STEP 5에서 `settings-template.json`의 `permissions.allow` 배열에 추가할 Bash 명령 권한들. STACK_BACKEND/STACK_USER_FE/STACK_ADMIN_FE/STACK_MOBILE/HARNESS 응답에 따라 합집합으로 추가.

---

## 언어·런타임 (STACK_* 매칭)

- **Python**: `"Bash(python *)"`, `"Bash(pip *)"`, `"Bash(pytest *)"`, `"Bash(uvicorn *)"`, `"Bash(poetry *)"`, `"Bash(uv *)"`
- **Node.js / React / Next / Vue / Angular / SvelteKit / NestJS**: `"Bash(npm *)"`, `"Bash(npx *)"`, `"Bash(yarn *)"`, `"Bash(pnpm *)"`
- **Java/Maven**: `"Bash(mvn *)"`, `"Bash(java *)"`
- **Java/Gradle / Kotlin**: `"Bash(./gradlew *)"`, `"Bash(java *)"`
- **Go**: `"Bash(go *)"`, `"Bash(gofmt *)"`, `"Bash(golangci-lint *)"`
- **.NET**: `"Bash(dotnet *)"`
- **Flutter**: `"Bash(flutter *)"`, `"Bash(dart *)"`
- **React Native / Expo**: `"Bash(expo *)"`, `"Bash(npx react-native *)"`
- **iOS 네이티브**: `"Bash(xcodebuild *)"`, `"Bash(pod *)"`
- **Android 네이티브**: `"Bash(./gradlew *)"`, `"Bash(adb *)"`
- **Docker**: `"Bash(docker *)"`, `"Bash(docker-compose *)"`, `"Bash(docker compose *)"`

## CI/CD (HARNESS 매칭)

- **GitHub Actions**: `"Bash(gh *)"`
- **GitLab CI**: `"Bash(glab *)"`
- **Jenkins**: `"Bash(jenkins-cli *)"`

## 린터·포매터 (Q5-A LINTER_* 매칭)

- **ESLint/Prettier**: `"Bash(eslint *)"`, `"Bash(prettier *)"`, `"Bash(npx eslint *)"`
- **Biome**: `"Bash(biome *)"`, `"Bash(npx biome *)"`
- **Black/Ruff/isort**: `"Bash(black *)"`, `"Bash(ruff *)"`, `"Bash(isort *)"`
- **Checkstyle/Spotless**: `"Bash(mvn checkstyle:check)"`, `"Bash(./gradlew spotlessApply)"`, `"Bash(./gradlew check)"`
- **golangci-lint**: `"Bash(golangci-lint *)"`
- **SwiftLint**: `"Bash(swiftlint *)"`
- **ktlint**: `"Bash(ktlint *)"`
- **dart format / flutter analyze**: `"Bash(dart *)"`, `"Bash(flutter analyze)"`

## 테스트 (Q5-A TEST_UNIT_* / TEST_E2E / TEST_PERF 매칭)

- **PyTest**: `"Bash(pytest *)"`
- **Jest/Vitest**: `"Bash(jest *)"`, `"Bash(vitest *)"`, `"Bash(npx jest *)"`, `"Bash(npx vitest *)"`
- **JUnit/Mockito**: 이미 Maven/Gradle allow에 포함
- **Go test**: `"Bash(go test *)"`
- **Cypress/Playwright**: `"Bash(cypress *)"`, `"Bash(playwright *)"`, `"Bash(npx cypress *)"`, `"Bash(npx playwright *)"`
- **JMeter/k6/Gatling**: `"Bash(jmeter *)"`, `"Bash(k6 *)"`, `"Bash(gatling *)"`
- **flutter_test**: `"Bash(flutter test *)"`

## 정적·보안 분석 (Q5-B STATIC_* 매칭)

- **SonarQube**: `"Bash(sonar-scanner *)"`, `"Bash(mvn sonar:sonar *)"`
- **Semgrep**: `"Bash(semgrep *)"`
- **Snyk**: `"Bash(snyk *)"`
- **OWASP Dependency-Check**: `"Bash(dependency-check *)"`
- **Trivy**: `"Bash(trivy *)"`
- **Grype**: `"Bash(grype *)"`
- **OWASP ZAP**: `"Bash(zap-cli *)"`, `"Bash(zap.sh *)"`
