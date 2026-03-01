# Post-Workout Report Template

> This template defines the standard output format for post-workout reports.  
> Fields in `[brackets]` are placeholders. Omit fields that don't apply to the activity type.

---

```
Data (last_updated UTC: [YYYY-MM-DDTHH:MM:SS])

[One-line summary of completed session(s) and key observation.]

Completed workout: [ActivityType] [WorkoutName]
Start time: [HH:MM:SS]
Duration: [XhYm] (planned [XhYm])
Distance: [XX.X] km
Power: [XXX] W avg / [XXX] W NP
Power zones: [XX]% Zone 1, [XX]% Zone 2
Grey Zone (Z3): [XX]%
Quality (Z4+): [XX]%
Session profile: [Classification]
HR: [XXX] avg / [XXX] max
HR zones: [XX]% Zone 1, [XX]% Zone 2
Cadence: [XX] avg
Decoupling: [X.XX]%
EF: [X.XX]
Variability Index: [X.XX] ([assessment])
Calories: [XXXX] kcal
Carbs used: [XXX] g
TSS: [XXX] (planned [XXX])

[Repeat block for additional sessions]

Weekly totals:
Polarization: Z1+Z2 [XX]%, Z3 [X]%, Z4+ [X]% — [Classification] (PI: [X.XX])
Durability: [X.XX]% 7d mean([X]) / [X.XX]% 28d mean([X]) ([trend])
EF: [X.XX] 7d mean([X]) / [X.XX] 28d mean([X]) ([trend])
TID 28d: [Classification] (PI: [X.XX]) — drift: [consistent/shifting/acute_depolarization]
TSB: [X.XX]
CTL: [XX.XX]
ATL: [XX.XX]
Ramp rate: [X.XX]
ACWR: [X.XX] ([assessment])
Recovery Index: [X.XX]
Hours: [XhYm]
TSS: [XXX]

Interpretation:
[2-4 sentences: compliance check, key quality metrics, load context, recovery note if applicable.]
```

---

## Rounding Convention

Round zone percentages to the nearest **whole number** (1%). The JSON data source carries precise values for detailed analysis. A few seconds in a zone is noise, not signal — report `0%` not `0.1%`.

## Field Notes

| Field | When to include | Notes |
|-------|----------------|-------|
| Distance | Cycling, running | Omit for SkiErg, strength |
| Power / Power zones | Activities with power data | Omit if no power meter |
| Grey Zone / Quality | Always for cycling | Highlights polarization compliance |
| Cadence | Cycling, running | Omit for SkiErg, strength |
| Decoupling | Sessions ≥ 1 hour | Key aerobic efficiency marker. Per-session scale (<5% good) per Friel/Coggan. Aggregate durability uses tighter scale (<3% good) |
| EF | Activities with power + HR | Aerobic efficiency (NP ÷ HR); track trend over like-for-like sessions. Absolute value is individual-dependent |
| Variability Index | Cycling with power | 1.00–1.05 = steady, >1.05 = variable. Assessment labels apply to steady-state only; omit label for interval sessions where high VI is expected |
| Carbs used | Sessions with power data | Omit if unavailable |
| Durability (weekly) | Aggregate decoupling 7d/28d | Steady-state sessions only (VI ≤ 1.05, ≥ 90min). Trend direction matters more than absolute value |
| EF (weekly) | Aggregate EF 7d/28d | Steady-state cycling only (VI ≤ 1.05, ≥ 20min). Trend direction matters more than absolute value |
| TID 28d (weekly) | 28d Seiler classification + drift | Shows whether acute TID matches chronic pattern. Always include drift label |
| Weekly totals | Always | Running totals through current day |

## Assessment Labels

| Metric | Good | Watch | Flag |
|--------|------|-------|------|
| Decoupling (per-session) | < 5% | 5–10% | > 10% |
| Variability Index | ≤ 1.05 | 1.05–1.10 | > 1.10 |
| ACWR | 0.8–1.3 | 1.3–1.5 | > 1.5 or < 0.8 |
| Grey Zone (Z3) | < 5% (base) | 5–10% | > 10% (base phase) |
| Durability (7d mean) | < 3% (good) | 3–5% (moderate) | > 5% (declining) |
| EF trend | improving/stable | — | declining |
| TID drift | consistent | shifting | acute_depolarization |

## Formatting Rule

- **Durations and sleep:** Always use `_formatted` fields from JSON (e.g., `sleep_formatted`, `duration_formatted`, `total_training_formatted`). Never convert decimal `_hours` fields to display format — the formatted values are pre-calculated from raw seconds and avoid rounding errors.
