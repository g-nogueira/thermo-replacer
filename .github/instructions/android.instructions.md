---
description: "Kotlin/Android specific grounding rules and patterns for the Genial-T31 Bridge app"
applyTo: "app/**/*.kt"
---

# Android / Kotlin Instructions

These rules apply to all Kotlin source files in the Genial-T31 Bridge Android app.

## Language & Style

- **Kotlin idioms:** Use `val` over `var`, data classes, sealed classes, extension functions, scope functions (`let`, `run`, `also`, `apply`) where they improve clarity
- **Coroutines:** Use `suspend` functions and structured concurrency. Prefer `Flow` for reactive streams. Never use `GlobalScope`.
- **Nullability:** Leverage Kotlin's null safety. Avoid `!!` — use `?.`, `?:`, or early returns instead
- **Naming:** `camelCase` for functions/properties, `PascalCase` for classes/interfaces, `SCREAMING_SNAKE_CASE` for constants

## Architecture Conventions

- **Single Activity:** `MainActivity.kt` hosts the Jetpack Compose UI
- **No Fragment usage** — Compose navigation only
- **Package structure:** `ble/`, `data/`, `worker/`, `ha/`, `health/`, `ui/` under `com.genialbridge`
- **DataStore** for preferences — no SharedPreferences
- **WorkManager** for background tasks — no AlarmManager or JobScheduler directly

## BLE Code Rules

- **Android native BLE only** — `BluetoothLeScanner` + `BluetoothGatt`. No third-party BLE libraries.
- **Always disconnect after polling.** Connect → read → disconnect. Never hold idle connections.
- **Handle `onConnectionStateChange` failures gracefully** — log and retry on next WorkManager cycle
- **All BLE hex values must match `Docs/FINDINGS.md`** — quote exact hex when implementing protocol code
- **BLE operations must run off the main thread** — use coroutines with appropriate dispatchers

## Jetpack Compose Rules

- **Material 3** components and theming
- **State hoisting:** UI components receive state as parameters, emit events via lambdas
- **Preview:** Add `@Preview` annotations for composable functions where practical
- **No side effects in composables** — use `LaunchedEffect`, `DisposableEffect`, `SideEffect` as needed

## Testing

- **Unit tests** under `app/src/test/` — JUnit 4, MockK, kotlinx-coroutines-test
- **Instrumented tests** under `app/src/androidTest/` — for Compose UI tests and BLE integration tests
- **Test naming:** backtick-quoted descriptive names: `` `parses temperature packet correctly` ``
- **BLE protocol tests:** Verify packet building and parsing against known hex values from `Docs/FINDINGS.md`

## Dependencies

Only use dependencies listed in [Docs/PROJECT_SPEC.md](../../Docs/PROJECT_SPEC.md) (Tech Stack section). Adding new libraries requires explicit user approval.

## Security

- **HA tokens** stored in DataStore (encrypted if available via EncryptedSharedPreferences fallback for tokens)
- **No hardcoded credentials** — all sensitive config comes from user settings
- **BLE permissions** requested at runtime with proper rationale UI
- **Network calls** validate URLs and use HTTPS where possible
