# Debounce (Code Time for JetBrains)

Coding-time tracking plugin for JetBrains IDEs. It collects editor activity, debounces it, and reports coding events to the [Code Time](https://codetime.dev) service — your coding time shows up right in the status bar.

Works with any IntelliJ Platform IDE (IntelliJ IDEA, PyCharm, WebStorm, GoLand, CLion, …) built on the 2024.3 platform or later.

## Features

- **Editor event collection** — captures document edits, caret moves, selection changes, visible-area scrolls, and editor creation via the global `EditorFactory` event multicaster.
- **5-second debounce** — events fired less than 5s apart are suppressed, so a burst of keystrokes produces a single `FILE_EDITED` event instead of one per keystroke.
- **Status bar widget** — a `⏰ CodeTime` widget polls `api.codetime.dev` every 60 seconds and shows your coding time as human-readable duration text (e.g. `2hrs 34mins`); supports three time ranges: total, last 24 hours, and today.
- **Settings page** — *Settings → Codetime Settings* stores your Code Time account token and status-bar time range. Until a token is set, the widget reads `Click to enter your token` and clicking it opens the settings page; with a valid token, clicking opens codetime.dev.

## Installation

- **From a release archive** — download the plugin zip from [Releases](https://github.com/Dyrk2020/Debounce/releases) (or build it yourself below), then **Settings → Plugins → ⚙ → Install Plugin from Disk…**, select the zip, and restart the IDE.
- **From the JetBrains Marketplace** — search for **CodeTime** (vendor CodeTime, plugin id `dev.codetime.codetime-jetbrains`) in **Settings → Plugins → Marketplace**, install, and restart. A marketplace release is published automatically by the `Release` GitHub Actions workflow whenever a release is published.

## Building from source

Requires JDK 17; use the bundled Gradle wrapper.

```bash
./gradlew buildPlugin     # plugin zip lands in build/distributions/
./gradlew verifyPlugin    # run the IntelliJ Plugin Verifier against IDEA Ultimate 2024.3
./gradlew publishPlugin   # publish to the Marketplace (needs PUBLISH_TOKEN env var)
```

## How it works

On project startup (`CodetimeStartupActivity`, a `ProjectActivity`), the plugin registers six listeners, each debounced to 5s: document edits → `fileEdited` (write), caret/selection/visible-area → `changeEditorSelection` / `changeEditorVisibleRanges` (read), editor creation → `fileCreated` (write), and file activation → `activateFileChanged` (read). Each qualifying event is serialized to JSON (`EventLog.kt`) and posted asynchronously with a Ktor CIO client to `https://api.codetime.dev/eventLog`, with your token in the `token` header; the status bar widget separately queries `https://api.codetime.dev/user/minutes?minutes=<range>`.

Reported fields include project name, file paths, file language, IDE name, platform and arch, timestamp, git origin URL and branch, plus the event type (`fileEdited`, `fileCreated`, `activateFileChanged`, `changeEditorSelection`, `changeEditorVisibleRanges`, …) and operation type (`read`/`write`) — file contents are never sent. The token is stored locally via the IDE's `PropertiesComponent` and sent only to `api.codetime.dev`; events are posted on a background dispatcher and never block the editor.

## Tech stack

Kotlin 2.0.20 · IntelliJ Platform Gradle Plugin 2.1.0 (IDEA Ultimate 2024.3) · Ktor client 3.0.0 (CIO) · kotlinx-serialization-json 1.7.3

## License

No license file is currently included in this repository. All rights reserved by the author unless a license is added.
