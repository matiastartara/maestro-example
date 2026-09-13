# 🧭 maestro-android

UI tests for the Wikipedia Android app (`org.wikipedia`), written with [Maestro](https://maestro.dev/). 🤖📱

## 🗂️ Project structure

```
maestro-android/
├── flows/              # Full end-to-end tests (the ones you actually run)
│   └── search_roma.yaml
├── subflows/            # Reusable per-screen steps (Maestro's "page objects")
│   ├── onboarding.yaml       # dismisses the welcome carousel and opens the search tab
│   ├── search_screen.yaml    # opens the search bar and types a term
│   └── article_page.yaml     # opens the first result and asserts the title
└── README.md
```

Maestro doesn't have "Page Object" classes like Selenium/Appium: the closest
equivalent is one subflow YAML per screen or action, composed from a flow via
`runFlow`. Each subflow still needs its own header (`appId` + `---`) even
though it's never run on its own.

A flow (`flows/search_roma.yaml`) sets the `appId`, environment variables
(`env`), and chains the subflows together:

```yaml
appId: org.wikipedia
name: "Search a term and open the first result"
env:
  SEARCH_TERM: "Roma"
---
- launchApp:
    clearState: true
- runFlow: ../subflows/onboarding.yaml
- runFlow: ../subflows/search_screen.yaml
- runFlow: ../subflows/article_page.yaml
```

✨ To add a new test (e.g. search a different term), the simplest way is to
create a new file in `flows/` that reuses the same subflows with a different
`SEARCH_TERM`.

## ⚙️ Installation

Maestro runs against an emulator/simulator or browser that's already running;
it doesn't install or launch that for you.

1. **Install the Maestro CLI** 🛠️

   ```bash
   curl -Ls "https://get.maestro.mobile.dev" | bash
   ```

   Verify with `maestro --version`. Requires Java (JDK 11+) installed.

2. **Android** 🤖: have Android Studio / SDK installed, with an emulator created
   in the Android Virtual Device Manager, or a physical device with USB
   debugging enabled. Confirm it's visible with `adb devices`.

3. **Install the app under test** 📲 (Wikipedia) on the emulator/device, either
   from the Play Store or by installing the APK with `adb install`.

## ▶️ Running the tests

With the emulator/device running:

```bash
# A single flow
maestro test flows/search_roma.yaml

# Every flow in the folder
maestro test flows/

# Inspect the current screen's hierarchy (useful for writing selectors)
maestro studio
```

Files under `subflows/` are never run directly: they're invoked by files in
`flows/` through `runFlow`.

## 🤝 Using the Maestro MCP with Claude

This project has the Maestro MCP server configured, which gives Claude tools
to inspect the device and run flows without leaving the chat:

- 📋 `list_devices` — lists available emulators/simulators/browser and their `device_id`.
- 🔍 `inspect_screen` — fetches the current screen's hierarchy (to find an element's real `id`/`text`).
- 📸 `take_screenshot` — captures the current screen.
- 🚀 `run` — runs inline YAML or files from `flows/` against a `device_id`.
- 📖 `cheat_sheet` — full reference of Maestro's flow syntax.
- ☁️ `run_on_cloud` / `get_cloud_run_status` / `list_cloud_devices` — to run on Maestro Cloud instead of locally.

Typical flow when asking Claude to write or fix a test:

1. Start the emulator/device yourself (Claude won't do this for you). 🖥️
2. Ask "use the maestro mcp to..." — Claude will list devices, inspect the
   screen to confirm real selectors (never guessing them from a screenshot),
   and run the flow to validate it. ✅
3. Changes land in the `.yaml` files under `flows/`/`subflows/`, and you can
   re-run them yourself with `maestro test` as shown above. 🔁
