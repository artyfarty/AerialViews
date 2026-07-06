# Build & deploy

How the app is built and how a debug build is deployed to a TV device for daily-driver use.
Paths, device IPs and signing identities here are **placeholders** — keep real ones out of the
repo (this is a public project).

## Build-variant matrix

From `app/build.gradle.kts`:

- Base `applicationId = com.neilturner.aerialviews`.
- **`debug` buildType** adds `applicationIdSuffix = ".debug"` → a debug build installs as a
  **separate** app (`…​.aerialviews.debug`) alongside any release/Play install, and is
  `isDebuggable = true`. It's signed with the machine's Android debug keystore.
- **Flavors:** `github`, `beta` (default, adds a `-betaN` version suffix), `googleplay`,
  `googleplaybeta`, `amazon`, `fdroid`. Flavors change signing config, version-name suffix, and
  `src/<flavor>/` overlays — **not** the applicationId. All except `fdroid` pull in Firebase.
- Release signing reads `signing/release.properties` / `signing/legacy.properties`; those files
  and the keystore are **gitignored** and not in the repo.

Consequence: `<flavor>Debug` variants all share the debug-keystore signature, so a rebuilt debug
APK can update an installed debug app in place (`adb install -r`) without losing its settings.

## Building

No Linux-native JDK is required if you build through the Android Studio toolchain. In short:

```
# JAVA_HOME → Android Studio's bundled JBR; ANDROID_HOME → your SDK
./gradlew :app:assembleGithubDebug        # or gradlew.bat on Windows
```

APK lands at `app/build/outputs/apk/<flavor>/<buildType>/`. `google-services.json` (upstream's,
already committed) is needed by the Firebase flavors; `secrets.properties` is optional and absent
by default (`loadProperties` tolerates it missing).

## Deploying to a TV over network ADB

The target is typically headless (no USB), so ADB-over-network is the channel:

```
adb connect <device-ip>:5555
adb install -r app/build/outputs/apk/github/debug/app-github-debug.apk
```

- Use `install -r` (not uninstall+install) so **SharedPreferences survive** — the server URL,
  API key, selected albums, and overlay layout are all stored there and are tedious to re-enter.
- The `.debug` app is `isDebuggable`, so its prefs can be inspected/backed up with
  `adb backup -noapk <pkg>` or, for debug installs, `adb run-as <pkg> cat shared_prefs/…`.
- A Play-Store-installed release copy (`…​.aerialviews`, no `.debug`) is a different applicationId
  and signature, so it and the sideloaded debug app coexist untouched.
- The screensaver/daydream can be pointed at the debug app's `DreamActivity`; trigger it for
  testing with the platform's screensaver-start intent.

## Verifying a build against a live server

When debugging Immich API behaviour, capture `adb logcat` while triggering the screensaver and
watch the `ImmichRepository` / OkHttp lines. It's fine to probe a live Immich instance read-only
to confirm response shapes rather than guessing (see [immich-provider.md](immich-provider.md)).
