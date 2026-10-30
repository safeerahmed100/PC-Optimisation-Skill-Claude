# Advanced Windows optimization

Use this file only after BIOS, drivers, Tier 1 Windows settings, and game settings are done and measured, and only when a specific symptom or measurement points here. "I watched a video" is not a symptom.

## Contents
- The five-line change card
- Entry gate checklist
- Backup and rollback procedure
- Situational (Tier 2) entries
- Advanced (Tier 3) entries
- Placebo and harmful list, with reasons
- Reviewing a user-supplied script or guide

## The five-line change card

Present every advanced change in this form so the user can decide with full information:

```
Change:    <what exactly, with setting name/path/value>
Benefit:   <what it targets and what you expect, for THIS system>
Risk:      <what could go wrong; security/anti-cheat/stability impact>
Evidence:  <Proven | Situational | Contested | Unverified, plus why>
Verify/Undo: <how to measure it, and the exact command/steps to revert>
```

If Evidence would honestly be "Unverified" and Risk is non-trivial, do not recommend the change. Offer to test it only if the user insists and the undo is clean.

## Entry gate checklist

Before recommending anything in Tier 2/3, confirm all of these (say which are missing):

- [ ] Baseline benchmark exists and is repeatable (`benchmarking.md`)
- [ ] BIOS/drivers/Tier 1 settings are done
- [ ] There's a concrete symptom or bottleneck finding the change addresses
- [ ] Restore point and registry export taken; original value recorded
- [ ] Change is compatible with the user's anti-cheat (`faceit-safety.md`)

## Backup and rollback procedure

- Restore point: `Checkpoint-Computer -Description "Before <change>" -RestorePointType MODIFY_SETTINGS` (elevated).
- Registry key export before editing: `reg export "HKLM\<path>" "$env:USERPROFILE\Desktop\backup-<name>.reg"`; to undo, double-click the `.reg` or `reg import <file>`. If a value didn't exist before, undoing means deleting it, so record "did not exist."
- Services: record original startup type (`Get-Service <name> | Select Name,StartType,Status`), undo with `Set-Service -Name <name> -StartupType <original>`.
- BCD (boot) settings: `bcdedit /export C:\bcd-backup` before any `bcdedit` change; undo with `bcdedit /import C:\bcd-backup` (or `bcdedit /deletevalue <name>`).
- Keep everything in the change log.

## Situational (Tier 2) entries

These can help on specific systems. A/B test each; keep only if the benchmark shows improvement beyond noise.

- **Hardware-accelerated GPU scheduling (HAGS):** can reduce latency or cause stutter depending on GPU/driver/game. Toggle in Settings → System → Display → Graphics → Default graphics settings. Test both states in the game. Undo: toggle back, reboot.
- **VBS / Memory Integrity (HVCI):** these add a virtualization layer that can cost a small, system- and game-dependent amount of performance. Turning Memory Integrity off reduces a real security protection (it blocks vulnerable/malicious kernel drivers). Only consider it if the audit shows it running **and** a measurement shows a meaningful loss **and** the user accepts the security trade-off **and** the anti-cheat allows it. Default recommendation: leave it as Windows configured it. Undo: Windows Security → Device security → Core isolation.
- **NIC power-saving features** (Energy-Efficient Ethernet, Green Ethernet, "allow the computer to turn off this device") on wired adapters: disabling can remove occasional latency spikes on some NICs. Device Manager → adapter → Advanced/Power Management. Low risk, easily undone; measure with ping jitter / in-game net graph rather than FPS.
- **Background apps and overlays:** disable ones the user doesn't need (Tier 1 in practice).
- **Mouse polling rate / USB port choice:** higher polling (1000–8000 Hz) lowers input latency but costs CPU; 4000/8000 Hz can reduce FPS on weaker CPUs. Test. Plug high-polling devices into ports directly on the motherboard rear I/O.
- **Windows scheduled tasks and updates timing:** schedule active hours and restarts so updates don't run mid-session. Do not disable Windows Update.

## Advanced (Tier 3) entries

Do these only with a measured symptom (e.g., DPC spikes in LatencyMon tied to a specific driver, or periodic frametime spikes correlated with a service).

- **Identifying the culprit first:** use LatencyMon or a frametime trace to find the offending driver/process, then address *that* (update/rollback the driver, disable the specific device or service, swap the USB port). Fixing the identified culprit beats any global tweak.
- **MSI (message-signaled interrupts) mode for GPU/NIC:** modern GPU drivers already enable it by default. Only check it if DPC latency analysis implicates the device. Undo via the same registry key.
- **Service changes:** only disable a service you've identified as causing a problem *and* understand. Never batch-disable "unnecessary services" from a list. Set to Manual rather than Disabled where possible; Manual is reversible by Windows on demand.
- **Timer resolution:** modern Windows manages this per-process; blanket tools or registry changes can raise power use and sometimes hurt. Only test if a measurement points to timer-related jitter, and A/B both directions.
- **Network registry tweaks** (TCP settings, Nagle/ACK-delay): Nagle and delayed-ACK affect TCP traffic; most competitive FPS games use UDP, so these typically do nothing for them. Treat as Unverified for CS2.

## Placebo and harmful list

When the user asks about one of these, explain *why* and offer a measurement instead of just saying no.

| Tweak | Verdict | Why |
|---|---|---|
| Disabling Defender / firewall / Windows Update for FPS | Harmful | Tiny gain, large security cost; anti-cheat and updates depend on them |
| Disabling Spectre/Meltdown mitigations | Harmful | Real exploit exposure for a small, CPU-dependent gain |
| HPET on/off, `useplatformclock`, `disabledynamictick` as universal fixes | Contested/placebo | Behavior differs by CPU and Windows build; often neutral or negative; measure before adopting |
| `NetworkThrottlingIndex`, `SystemResponsiveness` registry values | Placebo | Targets multimedia scheduling; no demonstrated benefit for modern game input latency |
| `TcpAckFrequency` / `TCPNoDelay` for UDP games | Placebo | Affects TCP, not the UDP traffic most shooters use |
| "Disable core parking" registry/power hacks | Mostly placebo on modern Windows | Scheduler already handles it; can break X3D scheduling |
| Disabling memory compression, Superfetch/SysMain, paging file | Placebo/negative | Can increase hitching; pagefile removal can crash games |
| "RAM cleaner" / memory-flushing apps | Placebo | Windows manages standby memory; flushing causes disk reloads |
| Third-party "game boosters" / all-in-one optimizers | Harmful | Opaque changes, bundled bloat, anti-cheat flags |
| Modified/"lite" Windows ISOs | Harmful | Missing components, update breakage, security and anti-cheat risk |
| Win32PrioritySeparation edits | Contested | Possible small effect; evidence is thin; low priority |
| Disabling all startup/services via msconfig "hide Microsoft services" mass-kill | Harmful | Breaks things users later can't trace |
| Setting game process to Realtime priority | Harmful | Can starve input/audio/system threads and freeze the PC |

## Reviewing a user-supplied script or guide

1. Ask the user to paste it (or the guide's list of steps). Don't run or recommend it blind.
2. Split it into individual actions. For each, produce a five-line card or a one-line placebo/harmful verdict.
3. Keep only actions that pass the entry gate. Rebuild a small, reversible subset, with its own undo.
4. Tell the user what you removed and why, since they'll often see the same tweak again elsewhere.
