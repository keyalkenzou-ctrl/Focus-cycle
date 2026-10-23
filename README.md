# FocusCycle — Android native starter

FocusCycle is a Kotlin + Jetpack Compose starter project. It demonstrates a transparent Accessibility Service that redirects away from TikTok, Instagram, and Chrome while the stored phase is `FOCUS`.

## Build
- Android Studio Ladybug or newer
- JDK 17
- Android SDK Platform 35
- Open this folder in Android Studio, let Gradle sync, then run on a real Android device (API 26+).

## Permissions / setup
1. Install and open FocusCycle.
2. Tap **Aktifkan Accessibility Service**.
3. In Android Settings > Accessibility, enable **FocusCycle app guard** and confirm the disclosure.
4. Return to FocusCycle and test with TikTok, Instagram, and Chrome installed.
5. To test the timer without waiting 30 minutes, use Android Studio's debugger or temporarily change `FOCUS_MS` / `FREE_MS` in `CycleStore.kt`, then rebuild.

## Important implementation limits
- Accessibility events are used only to inspect the foreground package name. The service does not retrieve window text or content.
- This is not a secure kiosk / device-owner implementation. Accessibility can be disabled, force-stopped, or restricted by Android/OEM battery management; the target app may appear briefly before redirection.
- Android has no general API that proves a device is “unused” while the screen is off. This starter treats an uninterrupted screen-off interval as the qualification proxy. It cannot detect all background activity, scheduled device work, external input, or every wake event.
- The sample records the screen-off timestamp in persistent preferences, but this starter does not install a permanent background monitor or exact alarm. The timer/UI refreshes while FocusCycle is running. If the process is killed or the device reboots, the uninterrupted qualification is reset on boot; don't assume a transition happens exactly on time while the app is not running.
- The free phase expires based on wall-clock time when the app/service next runs; it is not guaranteed to transition at the exact second if Android suspends the process.
- No app can reliably “block” Chrome/TikTok/Instagram through Accessibility alone. Users can revoke access, force-stop the app, use another browser/app variant, or use system settings. This app does not block emergency calls, Settings, or safety functions.
- Notification permission is declared but no notification is currently posted. It is not required for the basic UI.
- Package names can change by app/region; add verified package names if needed.
- For stronger enforcement, use Android Enterprise device-owner / lock-task APIs on managed devices, with explicit provisioning and user consent. Those APIs are not a drop-in solution for ordinary consumer-installed apps.

## Suggested tests
1. **Accessibility disabled:** open target apps; FocusCycle should not claim enforcement is active.
2. **Focus redirect:** enable the service and open a target app. Expect FocusCycle to be brought forward; a brief flash is possible.
3. **Allowed app:** open another app. It should not be redirected.
4. **Screen-off streak:** turn the screen off for 30 minutes while the app process remains available. Confirm the qualification logic after reopening.
5. **Interrupted streak:** turn screen on before 30 minutes; the streak should reset when FocusCycle resumes.
6. **Free expiry:** temporarily shorten `FREE_MS`, enter the free phase using debugger/test values, and verify phase refresh returns to FOCUS.
7. **Reboot / force-stop / revoke permission:** verify the UI does not claim tamper-proof enforcement. Re-enable Accessibility manually after reboot if Android disables it.
8. **Emergency access:** verify emergency dialing and system safety functions remain available.

## Project files
- `MainActivity.kt`: Compose status and settings instructions
- `CycleStore.kt`: persisted phase and timer calculations
- `FocusAccessibilityService.kt`: foreground-package redirect
- `BootReceiver.kt`: resets screen-off streak after boot
- `AndroidManifest.xml`: activity, service, boot receiver and permissions
