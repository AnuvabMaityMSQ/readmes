# WebSearch Moderation Engine: Development, Debugging & Engineering Guide

## 1. Build and Install

### 1.1 Gradle Build & APK Installation

- **Clean and assemble debug APK:**
`./gradlew :app:assembleDebug`
- **Install onto connected device (reinstall preserving data):**
`adb install -r -d app/build/outputs/apk/debug/app-debug.apk`
- **Alternative: Build and install directly in one step:**
`./gradlew installDebug`
- **Verify target package is installed:**
`adb shell pm list packages | grep com.guardian.child`

### 1.2 Running Targeted Unit Tests

- **Run all unit tests in the websafety module:**
`./gradlew :feature:websafety:testDebugUnitTest`
- **Run specific WebSearch coordinator & enforcement test suites:**
`./gradlew :feature:websafety:testDebugUnitTest --tests "com.guardian.child.websafety.websearch.coordinator.WebSearchModerationCoordinatorTest"`

---

## 2. Checking Logs

**Check chosen logs via custom TAGs:** `adb logcat -s <TAG>` (e.g., `adb logcat -s WebSearchLogger`)

**Check all WebSearch module logs:** `adb logcat -s WebSearchLogger:V WebParentalOverlay:V BrowserStateTracker:V WebSearchEnforcementMacro:V EduClassifierHolder:V WebSafetyPolicyStore:V WebSafetyInitializer:V GuardianAccessibility:V A11yRegistry:V`

| Logcat TAG | Key Events Logged |
|---|---|
| `WebSearchLogger` | Evaluation ASCII banners, SERP clicks captured, URL extractions, ML scores, policy decisions |
| `WebParentalOverlay` | WindowManager overlay presented/removed, "Go Back" button taps, back key event interception |
| `BrowserStateTracker` | State transitions (`OUTSIDE_BROWSER` -> `SERP` -> `REGULAR_WEBPAGE`), query parsing, URL bar reads |
| `WebSearchEnforcementMacro` | Cooldown throttling (`2000ms`), overlay presentation triggers, `GLOBAL_ACTION_BACK` execution |
| `EduClassifierHolder` | Background asset streaming, pre-warming duration, synchronous fallback loads |
| `WebSafetyPolicyStore` | SharedPreferences loading, remote policy updates, atomic state swaps |
| `WebSafetyInitializer` | Subsystem bootstrap, tracker & coordinator registrations into Accessibility pipeline |
| `GuardianAccessibility` | `GuardianAccessibilityService` lifecycle, service connection, active window rechecks |
| `A11yRegistry` | Interceptor registrations/unregistrations, service connected callbacks, event dispatches |

---

## 3. Deep Inspection: Node Tree Dumps & System State

### 3.1 Capturing and Inspecting Active Window Hierarchies
- **Dump current screen UI hierarchy to XML on device:** 
`adb shell uiautomator dump /sdcard/window_dump.xml`
- **Pull the XML file to your local development machine:** 
`adb pull /sdcard/window_dump.xml ./window_dump.xml`

### 3.2 System Service State Dumps (`dumpsys`)

- **Accessibility Framework State:** 
`adb shell dumpsys accessibility`
- **Active Window Overlays (Verify `TYPE_ACCESSIBILITY_OVERLAY`):** 
`adb shell dumpsys window windows | grep -E "Window #|com.guardian.child|TYPE_ACCESSIBILITY_OVERLAY"`
- **Foreground Activity & Browser Focus:** 
`adb shell dumpsys activity top | grep -E "ACTIVITY|package="`

---

