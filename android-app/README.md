# BPMN Mobile (Android)

An Android app for **viewing, editing, and simulating BPMN 2.0 diagrams**, built
from the same [bpmn-js](https://github.com/bpmn-io/bpmn-js) engine that powers
the Obsidian BPMN plugin in this repository.

It reuses the plugin's modeler configuration (see `../src/bpmnModeler.ts`) —
properties panel, color picker, create/append-anything, token simulation,
minimap, and grid — wrapped in a native Android WebView shell via
[Capacitor](https://capacitorjs.com/).

## Features

- **View** `.bpmn` files with pan/zoom (touch and drag-to-pan on canvas).
- **Edit** diagrams with the full bpmn-js modeler palette and context pad.
- **Simulate** process flow with the token-simulation module.
- **Properties panel** to edit element properties (toggle with the ⚙︎ button).
- **Open** `.bpmn`/`.xml` files via the native document picker.
- **Save / share** the diagram back out (`.bpmn`) via the Android share sheet.
- **Export** the diagram as **SVG**.
- **Autosave**: the working diagram is restored on next launch.

## Architecture

```
android-app/
├── index.html          # App shell + toolbar
├── src/
│   ├── main.js          # Boot, toolbar wiring, autosave
│   ├── modeler.js       # bpmn-js Modeler + feature modules (mirrors the plugin)
│   ├── files.js         # Native / web file open + save + share
│   ├── diagram.js       # Default empty diagram
│   └── style.css        # Mobile UI
├── capacitor.config.json
├── vite.config.js
└── android/            # Generated native Android (Gradle) project
```

The web app is built with Vite into `dist/`, which Capacitor copies into the
Android project's WebView assets.

## Prerequisites

- **Node.js** 18+ and npm
- **Android SDK** (via [Android Studio](https://developer.android.com/studio)),
  with `ANDROID_HOME` / `ANDROID_SDK_ROOT` set — required to compile the APK.
  The web build and `cap` scaffolding do **not** need the SDK; only the final
  Gradle step does.
- JDK 17 (bundled with recent Android Studio).

Project SDK levels: `minSdk 22`, `compileSdk 34`, `targetSdk 34`.

## Develop in the browser

Fast iteration on the UI/diagram logic without a device:

```bash
cd android-app
npm install
npm run dev        # open the printed http://localhost:5173 URL
```

In the browser, file open uses a normal file dialog and save triggers a
download (the native picker/share only exist on device).

## Build & run on Android

```bash
cd android-app
npm install
npm run build              # bundle the web app into dist/
npx cap sync android       # copy dist/ into the Android project + update plugins
npx cap open android       # open in Android Studio → Run ▶ on a device/emulator
```

### Build an APK from the command line

With `ANDROID_HOME` configured:

```bash
cd android-app
npm run apk:debug          # build web + sync + assembleDebug
# → android/app/build/outputs/apk/debug/app-debug.apk
```

Install it on a connected device:

```bash
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

For a release build, configure signing in `android/app/build.gradle` (or via
Android Studio's *Generate Signed Bundle / APK*) and run
`./gradlew assembleRelease` from the `android/` directory.

## Regenerating the native project

The `android/` directory is generated and checked in. If you ever need to
recreate it from scratch:

```bash
rm -rf android
npx cap add android
npx cap sync android
```

## Notes

- **App id:** `io.github.kimchaily.bpmnmobile` (change in
  `capacitor.config.json`, then re-run `npx cap sync android`).
- Native plugins in use: `@capacitor/filesystem`, `@capacitor/share`,
  `@capawesome/capacitor-file-picker`.
- Touch gestures: one-finger drag pans the canvas; element selection, the
  context pad, and the palette all work via pointer events. Pinch-zoom relies
  on the WebView; use the **⤢ Fit** button to re-fit the diagram.
