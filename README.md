<p align="center"><img src="docs/banner.png" alt="jev-use" width="100%"></p>

# jev-use

Voice and typed computer use for macOS. You say what you want. Jev picks the next on-screen action. macOS performs it. No screenshots: the app reads the screen through the Accessibility tree.

## New in this fork

- **Custom keyboard shortcut:** record your preferred key combination in Settings. It is saved across launches, with conflict handling that restores the previous shortcut.
- **Continuous hands-free mode:** speak, pause, and let the app act. Listening resumes after each command without holding a key.
- **Optional “Hey Jev” wake phrase:** activate hands-free mode with your voice, then keep speaking commands until you stop it.
- **Silence recovery:** an empty “No speech detected” timeout automatically restarts hands-free listening. Escape and Stop cancel pending restarts.

These features are available on the [`feature/custom-shortcut-hands-free` branch](https://github.com/maxxo-1/jev-use/tree/feature/custom-shortcut-hands-free). The `main` branch currently contains the original app code plus this updated guide. The changes have been offered upstream in [savka777/jev-use#2](https://github.com/savka777/jev-use/pull/2).

## Quick start

Requires macOS 14.2+ and Apple Command Line Tools or Xcode. Full Xcode is required to run the XCTest suite. No third-party package dependencies.

Clone the feature branch to build the updated app:

```sh
git clone --branch feature/custom-shortcut-hands-free https://github.com/maxxo-1/jev-use.git
cd jev-use
bash build.sh
open "$HOME/Applications/Desktop Voice.app"
```

In setup: save your TypeSafe API key (stored in the Keychain), then allow Accessibility, microphone and speech.

Hold **Control–Option–Space**, speak, then release to act. **Escape** cancels pending work. You can also type a command in **Settings and commands**, or run `scripts/say.sh "Open Finder"` from a shell.

### Change the keyboard shortcut

1. Open **Settings and commands** from the menu bar or the widget's gear icon.
2. Under **Choose your voice shortcut**, click **Change shortcut**.
3. Press the key combination you want to use.

Your choice is saved across launches and shown in the widget. Escape cancels recording. If another app has registered the combination, Desktop Voice reports the conflict and attempts to restore your previous shortcut. macOS-reserved combinations, Escape, and modifier-only shortcuts are not supported.

## Hands-free widget

Click **Start hands-free** in the widget. Speak a command and pause for about 1.5 seconds to submit it. The app waits for Apple's final transcript before acting, pauses its microphone while executing, then listens for your next command. Idle listening sessions renew automatically. Apple’s empty “No speech detected” timeout also restarts listening instead of switching hands-free mode off. Ordinary hands-free mode is off at launch. Enabling the wake-word setting starts background listening, including after app launch.

Click **Hands-free on · Stop**, press **Escape**, or open Settings to stop the active hands-free session. When wake-word activation is enabled, closing the widget or opening Settings returns to background wake-word listening. A microphone, recognition, or command error also stops it and shows the problem. Apple Speech may process audio online, as with hold-to-talk.

### Optional wake phrase

Enable **Invoke the widget by saying “Hey Jev”** in Settings. This immediately starts listening in the background while the app is running; no widget click is required. The saved option also starts wake-word listening after app launch once setup is ready.

Say “Hey Jev” on its own or followed by a command, such as “Hey Jev, open Finder”. The widget appears when the phrase is recognized, and hands-free stays active until you stop it. Closing the widget or opening Settings returns to background wake-word listening. **Stop** or **Escape** pauses all listening; use **Resume wake-word listening** in the menu bar to resume. Turn the setting off to disable automatic wake-word activation.

The phrase must begin the recognized utterance. Unrelated speech is discarded while waiting, without sending commands or screen context to Jev. Wake-word listening uses Apple Speech and may process audio online; it is not a dedicated offline wake-word engine.

## Examples

- "Open Obsidian, create a new note and type hello"
- "Go to youtube.com, search Rick Astley and play the first video"
- "Open 3 new tabs"
- "Scroll down three times"
- "Tile all the Brave windows so none are stacked"
- "In every Brave window, go to wikipedia.org and search for accessibility" — Jev works out the steps once, code repeats them in each window
- "Close the window", "Save", "New tab" — any item in the app's menu bar

## How it works

One loop, about 0.3–1.5 s per step:

1. **Read.** Walk the front app's Accessibility tree (~120 ms). Every element describes itself: what it is, its name, its value, where it sits, what it can do. No per-app code.
2. **Choose.** One request to Jev (`jev-latest`): the goal, the numbered targets, the last ten actions and their effects. Jev selects an operation and a target. It never generates free text; typed text is a span of your sentence.
3. **Act.** Press, select, type, menu, key, scroll, open, arrange windows.
4. **Check.** Read the screen again. Report the real effect. Repeat until DONE, BLOCKED or WAIT.

Low-confidence and destructive picks stop and ask instead of acting.

## What is sent

To `https://api.typesafe.ai/v1/systemone`: your command, the app and window names, the on-screen targets with their labels and values, and recent actions. Secure text fields are excluded. No screenshots. Speech uses Apple Speech.

Everything is logged locally: `log show --predicate 'subsystem == "local.jev-use"' --last 10m --info`

## Develop

Run these commands from the feature-branch checkout above. The focused checks cover shortcut preferences, silence recovery and cancellation, and wake-phrase matching. Live microphone and end-to-end command testing remain outstanding.

```sh
swift test        # requires XCTest from full Xcode
bash scripts/check-desktop.sh  # focused checks; Command Line Tools are sufficient
bash build.sh     # quit the app first
```
