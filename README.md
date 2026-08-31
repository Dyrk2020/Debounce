# Debounce

Coding-time tracking plugin for JetBrains IDEs. Listens to editor activity, debounces it, and reports coding events to the [Code Time](https://codetime.dev) service so your status bar shows how much you actually code.

Works with any IntelliJ Platform IDE (IntelliJ IDEA, PyCharm, WebStorm, GoLand, CLion, ...) on the 2024.3 platform or later. Built with Kotlin 2.0 and the IntelliJ Platform Gradle Plugin 2.1.

## Features

- **Editor event collection** — listeners on the global `EditorFactory` event multicaster capture document edits, caret moves, selection changes, visible-area scrolls, and editor creation
- **5-second debounce** — events fired less than 5 s after the previous one are suppressed, so a burst of keystrokes becomes a single `FILE_EDITED` event
- **Status bar widget** — `⏰ CodeTime` polls `api.codetime.dev` every 60 s and shows your coding time (total / last 24 h / today, e.g. `2hrs 34mins`)
- **Settings page** — *Settings → Codetime Settings*: account token + status-bar time range; the widget reads `Click to enter your token` until configured

## Build

```bash
./gradlew build           # build + test + verifyPlugin
./gradlew buildPlugin     # distributable zip under build/distributions
```

CI (`.github/workflows/build.yml`) validates the wrapper, runs `check` + `verifyPlugin`, builds the plugin artifact, runs the IntelliJ Plugin Verifier, and attaches the zip to a release.

## Source map

```
src/main/kotlin/dev/codetime/
  CodetimeStartupActivity.kt        plugin bootstrap, registers listeners + widget
  EventLog.kt / EventType.kt        event capture + 5 s debounce
  CodetimeStatusBarWidgetFactory.kt status bar widget (60 s polling)
  CodetimeConfigurable.kt           settings page (token, range)
  MinutesResponse.kt                API DTOs (kotlinx.serialization)
```
