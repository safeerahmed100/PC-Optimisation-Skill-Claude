# Final configuration report template

Use this exact structure for the Phase 6 deliverable. Fill every section with the user's real data; delete placeholders. If the user wants a file, produce a document in whatever format fits the session (see the session's file-creation guidance); otherwise present it in the reply.

```markdown
# PC Optimization Report: <game> profile
Date: <date>   Goal: <e.g., stable 1% lows, lowest latency>

## 1. System summary
CPU / GPU / RAM (rated vs actual) / Motherboard + BIOS version / Storage / Monitor + refresh / Windows build / GPU driver version / Anti-cheat

## 2. Key findings (ranked by impact)
1. <finding> — <impact>
2. ...

## 3. Results
| Config | Avg FPS | 1% low | 0.1% low | Max CPU temp | Notes |
|---|---|---|---|---|---|
| Baseline | | | | | |
| Final | | | | | |
Overall: <plain-language summary of what improved and by how much, including spread/noise>

## 4. Changes kept
| # | Change | Category | Tier | Original value | New value | Measured effect |
|---|---|---|---|---|---|---|

## 5. Changes tested and rejected
| Change | Result | Reason |
|---|---|---|
(Include placebo tweaks the user asked about, with the numbers or reasoning.)

## 6. Rollback plan
- Restore point name/date
- Registry backups and where they're stored
- Per-change undo steps (reference the change log)
- Last known-good GPU driver and BIOS version

## 7. Maintenance checklist
- Monthly: Windows Update, check GPU driver release notes (update only if beneficial), clean dust/filters
- After any driver or BIOS update: re-run the baseline benchmark and compare
- Re-check temps in a long session each season
- Keep startup apps trimmed; re-audit after installing new launchers/peripheral software
- Re-verify anti-cheat requirements before major Windows updates

## 8. Open items
Anything not resolved, with the next test to run.
```

Style notes: keep it factual and specific, no filler. Use "measured" for numbers you actually have and "expected" for anything else. If the headline result is "little to gain," say that and say why.
