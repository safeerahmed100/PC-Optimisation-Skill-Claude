# Counter-Strike 2 tuning (competitive / FACEIT)

CS2 is heavily **CPU-bound** at competitive settings on most modern GPUs, and highly sensitive to CPU cache and memory speed/latency. That shapes the priority order: CPU/RAM/thermals and background load matter more than GPU tricks. Patches change settings and defaults frequently, so when a detail matters (an option's name, a launch option's status, FACEIT rules), search the web for current information instead of trusting this file blindly.

## Contents
- What the user is optimizing for
- Reproducible benchmark for CS2
- Priority order
- Video settings logic
- Latency settings
- Launch options
- Config and input
- Network
- Common CS2 problems and likely causes

## What the user is optimizing for

Ask: target FPS vs. refresh rate, how bad the worst moments are (smokes, molotovs, many players, round start, spectating/demos), and whether feel (input latency) or visual clarity is the main complaint. For competitive play the usual answer is: stable high FPS with minimal dips and the lowest latency chain (mouse → game → GPU → monitor), at a visibility level the player is comfortable with.

## Reproducible benchmark for CS2

Use a repeatable scenario, not casual matchmaking. Options, in order of reliability:

1. A community **FPS benchmark map** from the Steam Workshop (search "CS2 FPS Benchmark"). It runs a scripted camera path with heavy scenes, giving comparable numbers across runs. Check the map's current instructions.
2. A **fixed offline route** on a map the user plays (e.g., load with bots, same spawn, same path, same duration, including a smoke/molotov moment).
3. Real match data from the in-game FPS overlay (`cl_showfps 1`) is useful for *symptoms* but not for A/B testing, because every match differs.

Run each configuration **3 times**, discard obvious outliers (alt-tab, background update), and compare medians. Capture with CapFrameX or PresentMon so you get 1% and 0.1% lows and the frametime graph, plus HWiNFO logging for temps/clocks/utilization. See `benchmarking.md` for the table format.

## Priority order

1. Thermals, memory speed/channel, background load, correct refresh rate (see audit).
2. Video settings (largest controllable effect on CPU-bound lows).
3. Reflex / latency settings.
4. FPS cap strategy.
5. Launch options and config (small, mostly hygiene).
6. Advanced Windows tweaks, only if a measured symptom remains.

## Video settings logic

Explain the trade-off per setting rather than handing over a "pro config":

- **Resolution / aspect ratio / display mode:** native or lower resolutions raise FPS mostly when GPU-bound; CPU-bound systems gain little. Stretched/4:3 is a visibility preference, not a performance trick. Use fullscreen (exclusive-style) for the lowest-latency path when available; test borderless only if alt-tabbing matters to the user, and compare latency/lows.
- **Global shadow quality, model/texture detail, shader detail, particle detail, ambient occlusion, high dynamic range, FidelityFX Super Resolution:** shadow and particle detail, AO, and some effects hit the CPU and GPU; lowering them typically improves lows. Texture detail mostly costs VRAM, not CPU; if VRAM is sufficient, medium/high textures are often free. Test in the smoke/molotov scenes since that's where dips live.
- **Anti-aliasing (MSAA 2x/4x/8x):** a large GPU and some CPU cost with a visibility benefit; start at 4x if the GPU has headroom, drop to 2x/off if frametimes suffer. Measure.
- **Boost Player Contrast:** a visibility option, usually enabled by competitive players; test its small cost.
- **Upscalers (FSR, etc.):** reduce image quality and are rarely the right fix for a CPU-bound game.
- **V-Sync:** off for competitive latency. Tearing is the trade-off; VRR (see below) is the alternative.

## Latency settings

- **NVIDIA Reflex Low Latency:** enable in CS2's advanced video settings on supported NVIDIA GPUs (Enabled, or Enabled + Boost if testing shows a benefit). It reduces render-queue latency, most noticeably when GPU-bound. On AMD GPUs, the Anti-Lag equivalent is the analogous option; check its current support status for CS2, since some Anti-Lag implementations have caused anti-cheat problems in other games. Verify with the vendor's documentation before enabling.
- **Driver control panel:** avoid stacking driver-level latency modes on top of the in-game Reflex setting; use one mechanism and measure. Set the power management mode to prefer maximum performance only if clocks are actually dipping in game.
- **G-Sync / FreeSync (VRR):** two valid strategies: (a) VRR on + V-Sync handling per vendor guidance + FPS cap just below refresh for smoothness without tearing, or (b) VRR off, uncapped/high FPS for the lowest possible latency with tearing. Let the user choose after trying both; say what you measured or what they felt, not what a video claimed.
- **FPS cap (`fps_max` / in-game limit):** uncapped can maximize responsiveness but may run the GPU/CPU hotter and make frametimes uneven when the engine's menus/loading hit very high rates. A cap at a stable level the system can always hold (for consistency) can improve 1% lows and thermals. Test: uncapped vs. a cap slightly below the system's average, comparing lows and frametime graphs. Don't pick a universal number.

## Launch options

Keep the list short and justified; many circulating lists are carried over from CS:GO and are inert in CS2.

- `-novid`: skips the intro video; harmless and saves a few seconds.
- `-console`: opens the console on start; convenience only.
- `+fps_max <n>`: sets a cap at launch (use only if the chosen cap strategy needs it).
- Options like `-high` (process priority), `-threads <n>`, `-nojoy`, `-tickrate`: treat as Unverified for CS2. Each needs a before/after and has no recommended default. `-high` can interact badly with background/audio threads; don't default to it.
- Anything that claims to "force" shader, engine, or API changes: check current Valve/community documentation first.

## Config and input

- **Raw input** enabled in game; mouse acceleration/pointer precision off in Windows.
- **Mouse polling rate and DPI:** choose for feel and a CPU you can afford (see polling note in `advanced-windows.md`); confirm sensor performance rather than chasing the max number.
- **Autoexec / `cfg` files:** only include commands the user understands. Use cloud settings carefully, since they can overwrite local changes and confuse A/B tests; record the config state when benchmarking.
- Keep a copy of the final config (and the Steam cloud setting) in the report.

## Network

CS2 uses UDP over Valve/FACEIT infrastructure. Prefer wired Ethernet, check ping/jitter/loss with the in-game net graph (`cl_showfps`/net_graph equivalent in current build) and a ping test, and fix the physical/ISP/router cause (Wi-Fi interference, router bufferbloat, NIC power-saving) before touching registry networking tweaks. Don't change `rate`/`cl_cmdrate`-style values unless current documentation says the defaults are wrong.

## Common CS2 problems and likely causes

| Symptom | First things to check |
|---|---|
| Low FPS despite strong GPU | CPU thermals/limits, RAM speed/XMP, background load, power plan, game on slow drive |
| Dips in smokes/molotovs | Shadow/particle/effects settings, CPU-bound limits, overlays, thermal throttling |
| Micro-stutter every few seconds | Background updaters, overlays, USB polling, NIC power-saving, driver regression, drive health |
| Fine in practice, bad in matches | Server load/network, FACEIT client overhead, 10-player scenes not matched in benchmark |
| Input feels delayed despite high FPS | VRR/V-Sync config, Reflex, fullscreen mode, monitor settings (overdrive/low-latency mode), render queue (GPU-bound) |
| Crashes / stutter after driver update | Roll back to a known-good driver; verify shader cache; check DDU only if needed |
