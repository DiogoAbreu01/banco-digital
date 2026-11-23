# Aurora Mobile — Expo SDK 58 compatibility review

## Problem and evidence

The Aurora mobile CI used Expo SDK 58 with several dependencies below the versions expected by the SDK's own `npx expo install --check` validator. The compatibility job failed even though the app's static type checks and tests were previously reported as passing.

The original CI diagnostic listed the following recommended versions. Direct dependencies were updated in `mobile/package.json`, and a **temporary GitHub Actions job** regenerated `mobile/package-lock.json` using npm on Node 24; the workflow deleted itself after committing the lockfile.

| Package | Before | Expo-expected version |
| --- | --- | --- |
| expo | ~58.0.17 | ~58.0.26 |
| expo-build-properties | ~58.0.11 | ~58.0.12 |
| expo-constants | ~58.0.13 | ~58.0.16 |
| expo-crypto | ~58.0.7 | ~58.0.8 |
| expo-font | ~58.0.9 | ~58.0.12 |
| expo-local-authentication | ~58.0.2 | ~58.0.3 |
| expo-secure-store | ~58.0.2 | ~58.0.3 |
| expo-status-bar | ~58.0.4 | ~58.0.5 |
| react-native | 0.88.0 | 0.88.1 |
| react-native-safe-area-context | ~5.10.1 | ~5.11.1 |
| react-native-screens | ~4.30.0 | ~4.30.1 |

## Verification gates

The lock-resolver action executes `npm install --package-lock-only --ignore-scripts`, a clean `npm ci --ignore-scripts` and `npx expo install --check` before creating its commit.

The repository CI must then independently pass `npm ci`, `npx expo install --check`, `npx expo-doctor`, TypeScript checks, mobile tests and Android/iOS `expo export` with example HTTPS API endpoints.

### Security remains a separate release blocker

The **mobile npm audit job must stay enabled and fail on high-severity vulnerabilities**. Upstream/transitive dependencies in Expo/Metro development and build tooling were previously flagged; SDK version alignment is not automatically a vulnerability fix. If the audit still fails, Aurora **must not** be labeled mobile-release-ready or fully green. A separate review of vulnerability chains and fixed upstream releases is required; no blind overrides of cryptographic or signing dependencies.

The app remains a **fictional digital banking simulator**, not a real banking product. Production secrets, live bank credentials, app-store signing and physical-device security validation are outside this change.
