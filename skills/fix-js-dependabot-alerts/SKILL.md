---
name: fix-js-dependabot-alerts
description: "JavaScript/Node.js 프로젝트의 GitHub Dependabot 보안 알림을 조회하고 취약한 transitive dependency를 자동으로 업데이트합니다. Yarn(Classic/Berry), npm, pnpm을 지원합니다. 'dependabot', '보안 취약점', 'security alert', 'vulnerability', '패키지 보안', 'npm audit', 'yarn audit', '취약한 패키지 업데이트', 'dependabot alert 해결' 같은 요청에 사용하세요. 사용자가 JS 패키지 보안 문제를 언급하거나 GitHub에서 Dependabot 알림을 받았다고 하면 반드시 이 스킬을 사용하세요."
---

# Fix JS Dependabot Alerts

GitHub Dependabot 보안 알림을 CLI로 조회하고, 취약한 패키지를 업데이트하는 워크플로우.

## 전제 조건 확인

작업을 시작하기 전에 반드시 아래 두 가지를 확인한다. 하나라도 실패하면 사용자에게 안내하고 중단한다.

### 1. gh CLI 확인

```bash
gh --version
```

설치되어 있지 않으면 사용자에게 `gh` 설치를 안내한다.

### 2. gh 인증 상태 확인

```bash
gh auth status
```

인증되지 않았으면 `gh auth login`을 안내한다.

### 3. 패키지 매니저 식별

현재 디렉토리에서 lock 파일을 확인하여 패키지 매니저를 식별한다:

| lock 파일 | 패키지 매니저 |
|-----------|-------------|
| `yarn.lock` | Yarn |
| `package-lock.json` | npm |
| `pnpm-lock.yaml` | pnpm |

Yarn인 경우 `yarn --version`으로 버전을 확인한다 (1.x Classic vs 2+ Berry — resolution 방식이 다름).

## 워크플로우

### Step 1: Dependabot 알림 조회

open 상태인 알림만 조회하고, 패키지별 패치 버전을 추출한다:

```bash
gh api 'repos/{owner}/{repo}/dependabot/alerts?state=open' \
  --jq '[.[] | {package: .security_vulnerability.package.name, patched: .security_vulnerability.first_patched_version.identifier, severity: .security_advisory.severity}] | unique_by(.package + .patched) | .[]'
```

`{owner}`와 `{repo}`는 `gh api`가 현재 git remote에서 자동 치환하므로 수동으로 지정할 필요 없다.

알림이 없으면 사용자에게 "보안 취약점 없음"을 알리고 종료한다.

결과를 사용자에게 테이블로 정리하여 보여준다.

### Step 2: 의존 경로 확인

각 취약 패키지가 어떤 경로로 설치되었는지 확인한다:

- **Yarn**: `yarn why <package>`
- **npm**: `npm ls <package>`
- **pnpm**: `pnpm why <package>`

direct dependency인 경우 — 해당 패키지를 직접 업데이트하면 된다.
transitive dependency인 경우 — resolution(override)으로 강제 업데이트해야 한다.

### Step 3: 패키지 업데이트

#### Direct dependency인 경우

패키지 매니저의 업데이트 명령어를 사용한다:

- **Yarn**: `yarn up <package>`
- **npm**: `npm install <package>@<version>`
- **pnpm**: `pnpm update <package>`

#### Transitive dependency인 경우

패키지 매니저별로 resolution/override를 설정한다:

**Yarn Berry (2+):**

`package.json`의 `resolutions` 필드에 추가:

```json
{
  "resolutions": {
    "<package>": "^<patched-version>"
  }
}
```

또는 CLI로:

```bash
yarn set resolution <package>@npm:<current-range> <patched-version>
```

**Yarn Classic (1.x):**

동일하게 `package.json`의 `resolutions` 필드를 사용한다.

**npm (9+):**

`package.json`의 `overrides` 필드에 추가:

```json
{
  "overrides": {
    "<package>": "^<patched-version>"
  }
}
```

**pnpm:**

`package.json`의 `pnpm.overrides` 필드에 추가:

```json
{
  "pnpm": {
    "overrides": {
      "<package>": "^<patched-version>"
    }
  }
}
```

### Step 4: 설치 및 검증

1. 패키지 재설치:
   - **Yarn**: `yarn install`
   - **npm**: `npm install`
   - **pnpm**: `pnpm install`

2. 업데이트 확인 — Step 2에서 사용한 명령어로 버전이 올라갔는지 검증한다.

3. 빌드 검증 — `package.json`의 `build` 스크립트를 실행하여 빌드가 깨지지 않았는지 확인한다.

### Step 5: 결과 보고

업데이트 결과를 테이블로 정리하여 사용자에게 보고한다:

| 패키지 | 이전 버전 | 업데이트 버전 | 심각도 |
|--------|----------|-------------|--------|
| example | 1.0.0 | 1.0.1 | high |

빌드 성공 여부도 함께 알린다.
