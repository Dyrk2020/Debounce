# Tickware (Code Time for JetBrains)

Coding-time tracking plugin for JetBrains IDEs. Tickware listens to your editor activity, debounces it, and reports coding events to the [Code Time](https://codetime.dev) service so you can see how much time you actually spend coding — right in your status bar.

Works with any IntelliJ Platform IDE (IntelliJ IDEA, PyCharm, WebStorm, GoLand, CLion, etc.) built on the 2024.3 platform or later.

## Features

- **Editor event collection** — registers listeners on the global `EditorFactory` event multicaster and captures document edits, caret moves, selection changes, visible-area scrolls, and editor creation.
- **5-second debounce** — every listener suppresses events fired less than 5s after the previous one, so a burst of keystrokes produces a single `FILE_EDITED` event instead of one per keystroke.
- **Status bar widget** — a `⏰ CodeTime` widget polls `api.codetime.dev` every 60 seconds and shows your coding time as human-readable duration text (e.g. `2hrs 34mins`). It supports three time ranges: total, last 24 hours, and today.
- **Settings page** — *Settings → Codetime Settings* lets you set your Code Time account token and choose the status-bar time range. Until a token is set, the widget reads `Click to enter your token` and clicking it opens the settings page; with a valid token, clicking opens codetime.dev.

## Installation

### From a release archive

1. Download the plugin zip from this repository's [Releases](https://github.com/Dyrk2020/Tickware/releases) page (or build it yourself, see below).
2. In your IDE, open **Settings → Plugins → ⚙ → Install Plugin from Disk…** and select the downloaded zip.
3. Restart the IDE.

### From the JetBrains Marketplace

Search for **CodeTime** (vendor: CodeTime, plugin id `dev.codetime.codetime-jetbrains`) in **Settings → Plugins → Marketplace**, install, and restart. A marketplace release is published automatically by the `Release` GitHub Actions workflow whenever a release is published.

## Building from source

Requirements: JDK 17, no local Gradle install needed (wrapper included).

```bash
./gradlew buildPlugin
```

The plugin zip lands in `build/distributions/`. Other useful tasks:

```bash
./gradlew verifyPlugin   # run the IntelliJ Plugin Verifier against IDEA Ultimate 2024.3
./gradlew publishPlugin  # publish to the Marketplace (needs PUBLISH_TOKEN env var)
```

## How it works

On project startup (`CodetimeStartupActivity`, a `ProjectActivity`), the plugin registers six listeners and maps IDE activity to event types:

| Listener | Debounce | Event type | Operation |
|---|---|---|---|
| `DocumentListener` | 5s | `fileEdited` | write |
| `CaretListener` | 5s | `changeEditorSelection` | read |
| `VisibleAreaListener` | 5s | `changeEditorVisibleRanges` | read |
| `SelectionListener` | 5s | `changeEditorSelection` | read |
| `EditorFactoryListener` | 5s | `fileCreated` | write |
| Second `DocumentListener` | 5s | `activateFileChanged` | read |

Each qualifying event is serialized to JSON and sent with a Ktor CIO HTTP client as a `POST` to:

```
https://api.codetime.dev/eventLog
```

with your token in the `token` header. The status bar widget separately queries:

```
https://api.codetime.dev/user/minutes?minutes=<range>
```

## Privacy: what is reported

Each event is a single JSON object with these fields (from `EventLog.kt`):

| Field | Meaning |
|---|---|
| `project` | Project name |
| `language` | File type name of the edited file (e.g. `kotlin`, `JAVA`) |
| `relativeFile` | Path of the file relative to the project root |
| `absoluteFile` | Absolute path of the file |
| `editor` | IDE name (e.g. `IntelliJ IDEA `) |
| `platform` | `os.name` (e.g. `Linux`) |
| `platformArch` | `os.arch` (e.g. `amd64`) |
| `eventTime` | Unix epoch milliseconds |
| `plugin` | Always `jetbrains` |
| `gitOrigin` | Git remote origin URL, if the file is inside a git repository |
| `gitBranch` | Current git branch name, if available |
| `eventType` | One of `activateFileChanged`, `editorChanged`, `fileAddedLine`, `fileCreated`, `fileEdited`, `fileRemoved`, `fileSaved`, `changeEditorSelection`, `changeEditorVisibleRanges` |
| `operationType` | `read` or `write` |

Requests carry your token in a `token` header; the token itself is stored locally via the IDE's `PropertiesComponent` and is only sent to `api.codetime.dev`. Events are posted asynchronously on a background dispatcher and never block the editor.

## Tech stack

- Kotlin 2.0.20, JVM target 17
- IntelliJ Platform Gradle Plugin 2.1.0, against IntelliJ IDEA Ultimate 2024.3
- Ktor client 3.0.0 (CIO engine) for HTTP
- kotlinx-serialization-json 1.7.3 for event payloads

## License

No license file is currently included in this repository. All rights reserved by the author unless a license is added.