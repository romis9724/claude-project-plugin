# .gitignore 패턴 풀

`/si-project:project-setup` STEP 2.5-2가 본 파일에서 컴포넌트별 패턴을 합집합으로 추출해 프로젝트 루트의 `.gitignore`로 작성. 마커 기반으로 Grep + Read 가능.

---

<!-- BLOCK: common -->
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editor
.vscode/
.idea/
*.swp
*~

# Env / Secret
.env
.env.*
*.secret
*.pem
*.key

# Logs / Coverage
*.log
coverage/
.nyc_output/

# Claude state (산출물 commit 안 함)
.claude/projects/
.claude/sessions/
<!-- /BLOCK: common -->

<!-- BLOCK: java-maven -->
# Backend Java/Maven
target/
*.class
*.jar
.mvn/
<!-- /BLOCK: java-maven -->

<!-- BLOCK: java-gradle -->
# Backend Java/Gradle
build/
.gradle/
*.jar
<!-- /BLOCK: java-gradle -->

<!-- BLOCK: python -->
# Backend Python
__pycache__/
*.pyc
.venv/
venv/
.pytest_cache/
.ruff_cache/
*.egg-info/
<!-- /BLOCK: python -->

<!-- BLOCK: node-backend -->
# Backend Node.js
node_modules/
dist/
<!-- /BLOCK: node-backend -->

<!-- BLOCK: go -->
# Backend Go
bin/
*.test
vendor/
<!-- /BLOCK: go -->

<!-- BLOCK: dotnet -->
# Backend .NET
bin/
obj/
*.user
<!-- /BLOCK: dotnet -->

<!-- BLOCK: frontend -->
# User FE / Admin FE (Node 기반 frontend)
node_modules/
dist/
.next/
.nuxt/
.svelte-kit/
out/
.cache/
.parcel-cache/
<!-- /BLOCK: frontend -->

<!-- BLOCK: mobile-flutter -->
# Mobile Flutter
.dart_tool/
build/
.flutter-plugins
.flutter-plugins-dependencies
ios/Pods/
ios/Flutter/Generated.xcconfig
<!-- /BLOCK: mobile-flutter -->

<!-- BLOCK: mobile-react-native -->
# Mobile React Native
node_modules/
ios/build/
android/build/
*.keystore
ios/Pods/
<!-- /BLOCK: mobile-react-native -->

<!-- BLOCK: mobile-ios-native -->
# Mobile iOS 네이티브
Pods/
*.xcworkspace/xcuserdata/
DerivedData/
*.xcuserstate
<!-- /BLOCK: mobile-ios-native -->

<!-- BLOCK: mobile-android-native -->
# Mobile Android 네이티브
*.iml
local.properties
.gradle/
build/
captures/
<!-- /BLOCK: mobile-android-native -->

---

## 적용 로직 (init이 수행)

1. `common` 블록은 항상 포함
2. `STACK_BACKEND` 텍스트 매칭:
   - "Maven" 포함 → `java-maven`
   - "Gradle" 또는 "Kotlin" 포함 → `java-gradle`
   - "Python" / "FastAPI" / "Django" 포함 → `python`
   - "Node" / "NestJS" / "Express" 포함 → `node-backend`
   - "Go" 단독 → `go`
   - ".NET" / "dotnet" → `dotnet`
3. `STACK_USER_FE_USE == 있음` 또는 `STACK_ADMIN_FE_MODE == separate` → `frontend`
4. `STACK_MOBILE_TYPE` 매칭:
   - "Flutter" → `mobile-flutter`
   - "React Native" / "RN" → `mobile-react-native`
   - "Swift" / "iOS" → `mobile-ios-native`
   - "Kotlin" / "Android" → `mobile-android-native` (Kotlin은 백엔드 가능성도 있음 — 컨텍스트 판단)

Grep으로 마커 위치 잡고 Read offset/limit로 해당 블록만 추출 → 합쳐서 `.gitignore` 작성.
