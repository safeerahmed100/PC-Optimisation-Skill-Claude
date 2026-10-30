# Benchmarking and verification

A change is only real if it beats run-to-run noise on a repeatable test. This file defines how to measure, how to read the numbers, and when to call something a win.

## Contents
- Principles
- Tools
- Test procedure
- Metrics to record
- Result table format
- Interpreting results
- Stability testing
- When measurement says "no difference"

## Principles

- **Same test every time:** same map/scenario, same route and duration, same settings except the one under test.
- **Same conditions:** fresh reboot or at least a consistent idle state, same background apps, same room/ambient temperature if possible, system warmed up (run a short warm-up pass first so cold-start boost doesn't flatter run 1).
- **Repeat:** at least 3 runs per configuration; compare medians and look at spread.
- **One variable at a time:** if two things changed, you can't attribute the result.
- **Record what you changed:** log setting, original value, new value, date.

## Tools

- **CapFrameX** or **PresentMon**: frametime capture, 1%/0.1% lows, frametime graphs.
- **HWiNFO64** (sensors): temps, clocks, power draw, throttling flags, CPU/GPU utilization. Log to CSV during runs.
- **LatencyMon**: DPC/ISR latency, only when investigating stutter or driver problems.
- **In-game:** `cl_showfps 1` for quick checks (not for A/B tests).
- **Latency (optional):** an end-to-end latency tool (e.g., NVIDIA LDAT or a high-speed camera) is the only real way to measure input latency; without one, say that latency conclusions are inferred from the settings used, not measured.

## Test procedure

1. Close non-essential apps; note what's running.
2. Reboot, wait a couple of minutes for startup activity to settle.
3. Warm-up run (discard).
4. Run the scripted scenario 3 times while capturing frametimes and logging sensors.
5. Save captures with a naming scheme: `<date>_<setting-changed>_run<n>`.
6. Paste the key numbers back (or export the summary).

## Metrics to record

| Metric | Why it matters |
|---|---|
| Average FPS | Overall throughput; least informative alone |
| 1% low FPS | Typical worst-case stutter experience |
| 0.1% low FPS | Rare hitches; very visible in competitive play |
| Frametime consistency | Variance or a graph; spikes feel worse than a lower average |
| CPU / GPU utilization | Shows whether the system is CPU- or GPU-bound |
| CPU / GPU temps and clocks | Detects thermal/power throttling and clock stability |
| RAM usage / pressure | Rules out paging or memory pressure |
| Stability | Crashes, freezes, WHEA errors, artifacts |

## Result table format

Present before/after to the user like this:

| Config | Avg FPS | 1% low | 0.1% low | CPU temp (max) | Notes |
|---|---|---|---|---|---|
| Baseline (median of 3) | | | | | |
| + XMP enabled | | | | | |
| + fps cap | | | | | |

Include the run-to-run spread (min–max of the 3 runs) so the user can see what counts as noise.

## Interpreting results

- A change counts as a **real improvement** only if the difference is clearly larger than the spread between runs *and* the frametime graph isn't worse. As a rough default, treat differences under about 2–3% on averages, or within the baseline's own min–max spread, as noise.
- Prefer improvements in **1% / 0.1% lows** and frametime consistency over average FPS gains.
- A gain that raises temperatures to throttling territory, or that comes with a crash, isn't a gain.
- If the result is mixed (average up, lows down), explain the trade-off and let the user choose according to their goal.

## Stability testing

After BIOS or advanced changes: extended real gameplay or a looped benchmark (an hour or more), plus a suitable stress test for the component changed. Check Event Viewer for WHEA-Logger, display driver reset (Event ID 4101), and application crashes. Don't consider a change kept until it's survived this.

## When measurement says "no difference"

Say so and revert if the change carries any risk or complexity. Reporting "tested, no measurable effect, reverted" is a valuable result: it tells the user which popular tweaks they can ignore. Record it in the final report's "Rejected" list with the numbers.
