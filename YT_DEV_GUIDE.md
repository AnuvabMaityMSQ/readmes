# YouTube Moderation Engine: Development, Debugging & Engineering Guide

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
- **Run specific YouTube test suites:**
`./gradlew :feature:websafety:testDebugUnitTest --tests "com.guardian.child.websafety.youtube.coordinator.YouTubeModerationCoordinatorTest"`

---

## 2. Checking Logs

**Check chosen logs via custom TAGs:** `adb logcat -s <TAG>` (e.g., `adb logcat -s YouTubeModCoordinator`)

**Check all YouTube module logs:** `adb logcat -s YouTubeModCoordinator:V PlaybackStateSensor:V ScreenStateTracker:V YouTubeContentExtractor:V IpcMetadataExtractor:V A11yIdExtractor:V CompoundCardExtractor:V TextSanitizer:V YouTubeEnforceMacro:V YouTubeParentalOverlay:V MiniplayerSweeper:V YouTubeHardeningMacro:V EduClassifierHolder:V WebSafetyPolicyStore:V WebSafetyInitializer:V A11yRegistry:V`

| Logcat TAG | Key Events Logged |
|---|---|
| `YouTubeModCoordinator` | Evaluation summaries, ML scores, decision blocks, macro triggers |
| `PlaybackStateSensor` | MediaSession bindings, playback detection, signature deduplication |
| `ScreenStateTracker` | Screen transitions (`BROWSE_FEED` -> `WATCH_PLAYER` -> `SETTINGS_PAGE`) |
| `YouTubeContentExtractor` | Routing between Layer 1, Layer 2, and Layer 3 |
| `IpcMetadataExtractor` | OS `MediaSessionManager` query matches (`title`, `artist`) |
| `A11yIdExtractor` | Resource ID matches (`video_title`, `channel_name`, Shorts regex) |
| `CompoundCardExtractor` | Heuristic node crawl for composite description cards |
| `TextSanitizer` | Regex noise stripping, view count removal, channel extraction |
| `YouTubeEnforceMacro` | Audio pausing, overlay triggers, back unwinding step-by-step |
| `YouTubeParentalOverlay` | Overlay view attached/dismissed, Back key interceptions |
| `MiniplayerSweeper` | Scheduled sweep attempts (80ms, 200ms...), close button clicks |
| `YouTubeHardeningMacro` | Onboarding automation steps, gear clicking, feed/autoplay toggling |
| `EduClassifierHolder` | Background asset reading, pre-warming duration, synchronous fallback |
| `WebSafetyPolicyStore` | SharedPreferences loading, remote updates, atomic swaps |
| `WebSafetyInitializer` | Subsystem startup and registration into Accessibility pipeline |
| `A11yRegistry` | Interceptor registration, callback dispatches |

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
- **MediaSession Inspection:** 
`adb shell dumpsys media_session`
- **Audio Focus & Stream Routing:** 
`adb shell dumpsys audio | grep -A 10 "Audio Focus"`
- **Foreground Activity & View Focus:** 
`adb shell dumpsys activity top | grep -E "ACTIVITY|package="`

---

## 4. Triggering the Settings Hardening Macro Manually
**Force-trigger the hardening macro:**
`adb shell am broadcast -a com.guardian.child.RUN_YOUTUBE_HARDENING_MACRO -p com.guardian.child --ez force true`

**Monitor execution:**
`adb logcat -s YouTubeHardeningMacro:V ScreenStateTracker:V`

---