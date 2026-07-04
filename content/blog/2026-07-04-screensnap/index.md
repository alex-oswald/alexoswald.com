---
title: "ScreenSnap: instantly switch between monitor configurations on Windows"
slug: screensnap-switch-windows-display-configurations
date: 2026-07-04
---

I have a TV connected to the PC in my office, and I kept running into the same annoyance: Xbox
mode (and honestly most "big picture" style experiences) only really behave on the *primary*
display. So every time I wanted to game on the couch, I had to dig into Windows display settings,
make the TV primary, flip the other monitor around, and then undo all of it when I wanted to get
back to work. That got old fast.

What I actually wanted was a single click to hot swap between two layouts:

- **Work** — my ultrawide on the desk is primary and the TV is disabled.
- **Xbox mode** — the TV is primary so Xbox mode lands on the right screen, and my ultrawide stays
  enabled as a secondary display.

Windows can obviously do both of those, it just makes you click through a settings page every
single time. So I wrote a little utility to save those layouts as presets and snap between them
instantly. I called it **ScreenSnap**.

Repo: [github.com/alex-oswald/ScreenSnap](https://github.com/alex-oswald/ScreenSnap)

## What is ScreenSnap

ScreenSnap is a lightweight Windows tray utility that switches between saved **display-configuration
presets**. A preset records, for every attached display, whether it's enabled, which one is primary,
its resolution, and its orientation. Once you've captured a couple of presets you can jump between
them from the taskbar tray menu or with a global keyboard shortcut, without opening Windows settings
at all.

![The ScreenSnap settings window: presets on the left, per-monitor enabled / primary / resolution / orientation options on the right](screenshot.png)

The screenshot above is basically my exact use case: a **Work mode** preset and a **TV gaming mode**
preset, with per-monitor toggles for my Dell ultrawide and the LG TV.

## Features

- **Taskbar tray icon** with a right-click menu listing every preset, plus **Settings** and **Exit**.
- **Per-monitor presets** — each preset stores, for every display, enabled/disabled, which one is
  primary, its resolution, and its orientation.
- **Global hotkeys** — cycle presets without leaving your game: **Ctrl + Alt + `+`** for the next
  preset and **Ctrl + Alt + `-`** for the previous one (main-row and numpad keys both work). The
  modifier chord is configurable, and there's an optional **Ctrl + Alt + 1…9** to jump straight to a
  specific preset.
- **Settings window** to capture the current layout as a preset, rename/reorder/delete presets,
  tweak per-monitor options, and apply a preset live.
- **Switch notifications** — a tray balloon confirms each switch and tells you when something failed,
  like a monitor that's no longer plugged in.
- **Start with Windows** (optional).
- **Minimal dependencies** — it talks to native Windows APIs through
  [CsWin32](https://github.com/microsoft/CsWin32) source-generated P/Invoke; there's no third-party
  UI or interop framework.

## How it works

The display engine uses the Windows **Connecting and Configuring Displays (CCD)** APIs
(`QueryDisplayConfig` / `SetDisplayConfig`) to enable and disable specific displays, set the primary
monitor, set each display's resolution and orientation, and position extended displays. The list of
selectable resolutions per monitor comes from `EnumDisplaySettingsEx`. Monitors are identified by
their stable device path, so a preset keeps working across reconnects and reboots instead of breaking
the moment Windows shuffles the display IDs around.

The tray icon is a classic `Shell_NotifyIcon` living on a hidden message-only window, and the global
hotkeys are registered with `RegisterHotKey` on that same window. There's no global keyboard hook, so
it coexists cleanly with Windows' own shortcuts. (My original idea was `Win + S`, but Windows reserves
that for Search, so the cycle hotkeys it is.)

## Getting started

Grab the latest installer from the
[**Releases**](https://github.com/alex-oswald/ScreenSnap/releases) page — there's an `x64` MSI for
most PCs and an `arm64` MSI for ARM devices. The installer is self-contained (it bundles the .NET and
Windows App SDK runtimes) and installs per-user, so there's no admin prompt and nothing else to
install.

The installers are code-signed, so they install cleanly without any SmartScreen "unverified
publisher" warnings.

Once it's running, look for the ScreenSnap icon in the notification area (you may have to expand the
tray overflow). To create your first preset:

1. Arrange your displays the way you want using Windows.
2. Open **Settings → Add current**, and give the preset a name.
3. Repeat for each layout you use.

After that, switch between them from the tray menu or with **Ctrl + Alt + `+`/`-`**. For me that's
one hotkey to drop into Xbox mode on the TV and one to snap right back to my desk.

## Under the hood

For the curious, ScreenSnap is built with **WinUI 3** / Windows App SDK on **.NET 10**, in C#. Native
interop goes through **Microsoft.Windows.CsWin32**, and release builds are compiled with **Native
AOT** into self-contained MSIs (built by a GitHub Actions workflow and packaged with the WiX Toolset).
The solution is split into a UI-agnostic `ScreenSnap.Core` display engine and the `ScreenSnap` tray
app, so the core knows nothing about WinUI or how the app is deployed.

It's [MIT licensed](https://github.com/alex-oswald/ScreenSnap/blob/main/LICENSE) and open on GitHub —
issues and PRs welcome. If you've got a TV bolted to your battlestation like I do, give it a try.
