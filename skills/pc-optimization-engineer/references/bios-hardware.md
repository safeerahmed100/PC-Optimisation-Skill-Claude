# BIOS and hardware

Hardware-level fixes cap everything above them, so do these first. Goal: a stable, cool, fully-clocked system, not an overclock.

## Contents
- Before entering BIOS
- Tier 1 settings
- Tier 2 settings
- Off-limits without explicit justification
- Thermals and fans
- Stability verification

## Before entering BIOS

1. Record the current BIOS version (from the audit) and the motherboard model.
2. Check the board vendor's support page for newer BIOS. Read the release notes: update for stability/compatibility/CPU microcode fixes, not just because a number is higher. If the system is stable and notes are irrelevant, an update is optional. Never update on an unstable power situation; never interrupt a flash.
3. Tell the user to photograph each BIOS screen they're about to change (or note values) so changes can be undone. Remind them that "Load Optimized Defaults" is the universal fallback.
4. BIOS layouts differ by vendor (ASUS, MSI, Gigabyte, ASRock) and version. Give the setting's name and typical location, but ask the user to describe or screenshot what they see if they can't find it, rather than guessing menu paths.

## Tier 1 settings

- **XMP / EXPO / DOCP:** enable the memory profile. Verify afterward with the audit's `ConfiguredClockSpeed`. If the system won't boot or crashes, try the other profile or a one-step-lower speed; don't manually raise voltage as a first reaction.
- **Dual-channel check:** DIMMs in the slots the manual specifies (typically A2/B2 for two sticks).
- **Boot mode:** UEFI, with Secure Boot enabled and TPM (fTPM/PTT) enabled. Needed for Windows 11 and anti-cheat (see `faceit-safety.md`).
- **Fan curves:** set a sensible curve so the CPU/GPU hold boost clocks without sounding like a jet. Raising a too-timid default curve is a safe, effective fix for thermal throttling.
- **Monitor connection:** confirm the display is on the GPU, not the motherboard output (a common mistake after a build).

## Tier 2 settings (test, don't assume)

- **Resizable BAR / Above 4G Decoding:** usually helps or is neutral; in a few games it's slightly negative. For CS2, benchmark with it on vs. off if the user cares; leave on by default if no difference. Requires UEFI/CSM-off boot and a compatible GPU/driver. Verify state in the GPU control panel or GPU-Z.
- **CPU boost / power limits (PBO on AMD, PL1/PL2 on Intel):** the right move is usually to make sure the board isn't *under*-limiting the CPU or *over*-feeding it into thermal throttling. Check actual sustained clocks and temps under a game load (HWiNFO) before touching anything. Intel 13th/14th-gen K-series owners should be on a current microcode BIOS (voltage-instability fixes) and should use Intel-default power profiles rather than "unlimited" board defaults.
- **AMD X3D CPUs:** install the AMD chipset driver (it includes the V-Cache scheduling driver). For dual-CCD X3D parts, make sure Windows Game Mode is on and Xbox Game Bar is not removed, because the scheduler uses it to detect games and place threads on the V-Cache cores. Don't hand-edit affinity before confirming the game isn't already landing on the right cores.
- **C-states / SMT / E-cores:** disabling these is common folklore. Treat as an A/B test with a specific hypothesis (e.g., measured clock ramp issue); don't recommend as a default.
- **PCIe link speed:** only if the GPU is running below its rated link (check in GPU-Z render test).

## Off-limits without explicit justification

Manual CPU/RAM voltage edits, aggressive memory timing/subtimings tuning, all-core overclocks, undervolt curves, LLC changes, disabling security features in BIOS. If the user wants to go there, treat it as a separate project: agree on a target, a stability test plan (e.g., multi-hour memory test, game + stress loop), and a revert path before touching anything. Do not offer these as casual "free FPS."

## Thermals and fans

Log temps during a real game session (HWiNFO sensors, "min/max/avg" columns). Typical concern thresholds, to be judged against the specific CPU/GPU's rated limits: sustained CPU at its thermal limit during games; GPU hot spot at its throttle point; VRM temps flagged. Fixes in order: dust/airflow/case fan orientation → fan curve → cooler mounting/paste → only then power limit trade-offs.

## Stability verification

A faster setting that crashes once a week is not optimized. After BIOS changes, verify with: a memory test appropriate to the change (e.g., a few passes of an established RAM stress tool), a GPU/CPU load, and an extended real game session. Watch for WHEA errors in Event Viewer (Windows Logs → System, source WHEA-Logger), a classic sign of unstable CPU/RAM/PCIe settings.
