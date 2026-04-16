---
name: android-dev
description: "Build, test, lint, and run the Genial-T31 Bridge Android app. Use when running Gradle commands, managing emulators, using ADB, or following the TDD Red-Green-Refactor loop."
---

# Android Development Skill

This skill provides the standard commands and workflows for building, testing, and running the Genial-T31 Bridge Android app.

## Build Commands

```powershell
# Full debug build
./gradlew assembleDebug

# Full release build
./gradlew assembleRelease

# Build without running tests
./gradlew assembleDebug -x test

# Clean build
./gradlew clean assembleDebug

# Check if project compiles (no APK output)
./gradlew compileDebugKotlin
```

## Test Commands

```powershell
# Run all unit tests
./gradlew test

# Run unit tests for debug variant
./gradlew testDebugUnitTest

# Run a specific test class
./gradlew testDebugUnitTest --tests "com.genialbridge.ble.GenialProtocolTest"

# Run a specific test method
./gradlew testDebugUnitTest --tests "com.genialbridge.ble.GenialProtocolTest.parsesTemperaturePacket"

# Run connected (instrumented) tests on device/emulator
./gradlew connectedDebugAndroidTest

# Run tests with verbose output
./gradlew test --info
```

## Lint & Static Analysis

```powershell
# Run Android lint
./gradlew lintDebug

# Run lint and generate HTML report
./gradlew lintDebug --continue
# Report at: app/build/reports/lint-results-debug.html

# Run detekt (if configured)
./gradlew detekt
```

## ADB Commands

```powershell
# List connected devices
adb devices

# Install debug APK
adb install -r app/build/outputs/apk/debug/app-debug.apk

# Uninstall app
adb uninstall com.genialbridge

# View app logs (filtered)
adb logcat -s "GenialBridge"

# View BLE-specific logs
adb logcat -s "BluetoothGatt" "BluetoothAdapter" "GenialBleClient"

# Clear app data
adb shell pm clear com.genialbridge

# Grant runtime permissions (useful for testing)
adb shell pm grant com.genialbridge android.permission.BLUETOOTH_SCAN
adb shell pm grant com.genialbridge android.permission.BLUETOOTH_CONNECT
adb shell pm grant com.genialbridge android.permission.ACCESS_FINE_LOCATION
```

## Emulator Commands

```powershell
# List available AVDs
emulator -list-avds

# Start emulator
emulator -avd <avd-name>

# Start emulator with BLE support (requires API 31+)
emulator -avd <avd-name> -feature Bluetooth
```

> **Note:** BLE testing on emulators is limited. Prefer a physical device for BLE-related work.

## TDD Red-Green-Refactor Loop

### 1. RED — Write a failing test

```kotlin
@Test
fun `parseTemperature extracts correct value from packet`() {
    val packet = byteArrayOf(
        0xA6.toByte(), 0x09, 0x01,
        0x0A, 0x44,  // T1 = 0x0A44 = 2628 → 26.28°C
        0x0A, 0x44,  // T2
        0x05, 0x14, 0x05, 0x0B,  // timestamp
        0xCF.toByte(), 0x6A
    )
    val result = GenialProtocol.parseTemperature(packet)
    assertEquals(26.28, result.temperature1, 0.01)
}
```

Run: `./gradlew testDebugUnitTest --tests "...parseTemperature*"` → should FAIL

### 2. GREEN — Write minimal code to pass

Implement just enough in `GenialProtocol.kt` to make the test pass.

Run: `./gradlew testDebugUnitTest --tests "...parseTemperature*"` → should PASS

### 3. REFACTOR — Clean up while keeping tests green

Run full test suite: `./gradlew test` → all should PASS

### Self-Verification Checkpoint

Before each commit, run:
```powershell
./gradlew clean assembleDebug test lintDebug
```

All three must pass. If lint has warnings, review them — fix any that are severity `error` or `warning`.

## Gradle Dependency Management

Dependencies are declared in `app/build.gradle.kts`:

```kotlin
dependencies {
    // Jetpack Compose
    implementation(platform("androidx.compose:compose-bom:2024.01.00"))
    implementation("androidx.compose.material3:material3")

    // WorkManager
    implementation("androidx.work:work-runtime-ktx:2.9.0")

    // Health Connect
    implementation("androidx.health.connect:connect-client:1.1.0-alpha07")

    // Networking (for HA webhook)
    implementation("com.squareup.okhttp3:okhttp:4.12.0")

    // DataStore
    implementation("androidx.datastore:datastore-preferences:1.0.0")

    // Testing
    testImplementation("junit:junit:4.13.2")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.8.0")
    testImplementation("io.mockk:mockk:1.13.9")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
}
```

> **Important:** Only add dependencies listed in [Docs/PROJECT_SPEC.md](../../../Docs/PROJECT_SPEC.md). New libraries require explicit user approval.

## Project Structure Reference

```
app/src/main/java/com/genialbridge/
├── MainActivity.kt
├── ui/            # Jetpack Compose screens
├── ble/           # BLE client + protocol
├── data/          # DataStore, data classes
├── worker/        # WorkManager jobs
├── ha/            # Home Assistant client
└── health/        # Health Connect integration
```
