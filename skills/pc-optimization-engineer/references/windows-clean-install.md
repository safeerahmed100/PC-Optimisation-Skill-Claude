# Fresh Windows installation

Goal: a clean, current, secure baseline. Most of the achievable gain on a typical gaming PC lives in this file and in the BIOS file; the exotic tweaks come later, if ever.

## Contents
- Install order
- Drivers
- Windows settings (Tier 1)
- Startup and background load
- Debloat (conservative)
- Security: leave it on
- Restore point and logging

## Install order

0. **Windows version:** Windows 11 is the recommended target. Windows 10 reached end of support in October 2025 (security updates only through Microsoft's Extended Security Updates program), so for a new install on supported hardware choose Windows 11, and for an existing Windows 10 PC explain the options. Check the user's anti-cheat requirements before deciding.
1. **Install Windows from official Microsoft media** (Media Creation Tool / ISO). Avoid "lite" or modified ISOs: they remove components, break updates, and can trip anti-cheat.
2. **First boot:** connect to the internet, run Windows Update to completion (reboot as needed, repeat until clean). This also pulls a baseline GPU driver.
3. **Motherboard/chipset drivers:** from the board or chipset vendor (AMD chipset package, or Intel chipset/ME/driver bundle as applicable). Add LAN/Wi-Fi/audio drivers from the board vendor only if Windows' versions are missing or problematic.
4. **BIOS update and settings:** see `bios-hardware.md` (ideally done before the final OS setup, but can be done now).
5. **GPU driver:** download from NVIDIA/AMD/Intel directly. A normal install over a fresh system is fine. Use DDU (Display Driver Uninstaller, in Safe Mode) only when migrating between GPU vendors or diagnosing a driver problem; it isn't a routine step. Skip bundled extras you don't need (e.g., optional audio/3D/GeForce Experience components) if the user prefers a lean install.
6. **Reboot, then take a restore point** (see below) before further changes.

## Drivers

Prefer the current stable driver from the vendor over beta unless the game has a specific fix in the release notes. If a new driver introduces stutter or regressions, roll back and record the version. Keep a note of the "known-good" driver version in the final report.

## Windows settings (Tier 1)

- **Display:** set the monitor to its native resolution and **highest refresh rate** (Settings → System → Display → Advanced display). It's surprisingly common for a 240 Hz monitor to sit at 60 Hz.
- **Power plan:** Windows 11 desktops are generally fine on Balanced or the Performance-oriented plan; confirm the CPU actually boosts under game load (HWiNFO). For laptops, plug in and use the performance mode. Don't import "Ultimate Performance" or custom `.pow` files from random sources; if a user wants one, `powercfg /duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61` creates the built-in Ultimate Performance plan, and the effect should be measured, not assumed. On AMD X3D systems, follow the vendor's guidance (chipset driver installed; Balanced is typically recommended so the scheduler logic works as designed).
- **Game Mode:** On. (Needed for the X3D scheduling behavior noted in `bios-hardware.md`; on other systems it's neutral-to-helpful.)
- **Mouse:** Settings → Bluetooth & devices → Mouse → Additional mouse settings → Pointer Options: **Enhance pointer precision off**. In-game raw input handles the rest. This is a consistency/feel fix, not an FPS one.
- **Game capture / overlays:** turn off Xbox Game Bar *recording* and unused overlays (Discord, Steam, GeForce) if the user doesn't use them. Do **not** uninstall Game Bar itself on X3D systems.
- **Notifications / Focus Assist:** quiet during gaming to avoid focus loss.
- **Storage Sense / indexing:** leave defaults; if a game library is on a secondary drive, exclude it from heavy third-party scanners.

## Startup and background load

Use Task Manager → Startup apps, and the audit's startup/process list. Disable (don't uninstall yet) launchers, updaters, RGB suites, OEM utilities, cloud sync clients, and chat apps the user doesn't need running during play. Each is small, but together they add background CPU wakeups and memory pressure. Re-test after; if a disabled item breaks something (e.g., a peripheral's macro software), re-enable it.

## Debloat (conservative)

Remove clearly unused preinstalled apps through Settings → Apps (OEM trialware, promoted games, etc.). Do not strip system components, Microsoft Store dependencies, Edge WebView2, or Windows Defender, and do not run aggressive debloat scripts (see the never-list in `SKILL.md`). Gains from removing app packages are mostly disk space and tidiness, not FPS. Set that expectation.

## Security: leave it on

Keep Windows Defender (real-time protection), firewall, Secure Boot, TPM, and Windows Update enabled. Typical-use performance cost is small. A game-folder exclusion for Defender is a more defensible compromise than disabling it, but even that is a judgment call to mention with its risk, not a default. VBS/Memory Integrity is a separate decision covered in `advanced-windows.md` and constrained by `faceit-safety.md`.

## Restore point and logging

Before moving on from clean install:

```powershell
Enable-ComputerRestore -Drive "C:\"
Checkpoint-Computer -Description "Pre-optimization baseline" -RestorePointType MODIFY_SETTINGS
```

(Elevated prompt. Windows limits restore-point frequency by default; if it declines, that's fine — note the existing latest restore point.) Start the **change log** now: date, what changed, original value, new value, measured effect. The final report is built from it.
