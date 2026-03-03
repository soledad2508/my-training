# Agentic — Write Planned Workouts to Intervals.icu

For AI platforms with code execution (OpenClaw, Claude Code, Claude Cowork, etc.). Chat-only users cannot use this.

## push.py

Validates and POSTs planned workouts to the Intervals.icu calendar using the same API credentials as `sync.py`.

### Credentials

Same sources as sync.py, checked in order:

1. **CLI args:** `--athlete-id`, `--api-key`
2. **Config file:** `.sync_config.json` (same file sync.py creates via `--setup`)
3. **Environment:** `ATHLETE_ID`, `INTERVALS_KEY`

If the athlete already uses sync.py, credentials are already set up.

### Single Workout (CLI)

```bash
python push.py \
  --name "Sweet Spot 3x15" \
  --date 2026-03-05 \
  --type Ride \
  --description "- 15m 55%\n\n3x\n- 15m 88-92%\n- 5m 55%\n\n- 10m 50%" \
  --duration 85 \
  --tss 75 \
  --target POWER
```

### Multiple Workouts (JSON file)

```bash
python push.py --json week.json --name ignored --date ignored
```

`week.json`:
```json
[
  {
    "name": "Endurance Z2",
    "date": "2026-03-05",
    "type": "Ride",
    "description": "- 10m ramp 50%-65%\n- 90m 65-75%\n- 10m ramp 65%-50%",
    "duration_minutes": 110,
    "tss": 70,
    "target": "POWER"
  },
  {
    "name": "VO2max 5x4",
    "date": "2026-03-07",
    "type": "Ride",
    "description": "- 15m ramp 50%-75%\n\n5x\n- 4m 106-120%\n- 3m Z1\n\n- 10m 50%",
    "duration_minutes": 60,
    "tss": 85,
    "target": "POWER"
  }
]
```

### Python Import (agent calls directly)

```python
from push import IntervalsPush

pusher = IntervalsPush(athlete_id="i123456", api_key="abc123...")

result = pusher.push_workout({
    "name": "Tempo 2x20",
    "date": "2026-03-06",
    "type": "Ride",
    "description": "- 15m 55%\n\n2x\n- 20m 76-90%\n- 5m 55%\n\n- 10m 50%",
    "duration_minutes": 75,
    "tss": 80,
    "target": "POWER",
})

if result["success"]:
    print(f"Created {result['count']} event(s)")
else:
    print(f"Failed: {result['error']}")
```

### Workout Fields

| Field | Required | Type | Notes |
|---|---|---|---|
| `name` | Yes | string | Workout name shown on calendar |
| `date` | Yes | string | `YYYY-MM-DD`, today or future only |
| `type` | No | string | `Ride` (default), `Run`, `VirtualRide`, `Swim`, etc. |
| `description` | No | string | Intervals.icu workout syntax (see below) |
| `duration_minutes` | No | number | Planned total duration including warmup/cooldown |
| `tss` | No | number | Planned training stress score |
| `target` | No | string | `POWER`, `HR`, or `PACE` |
| `category` | No | string | `WORKOUT` (default), `RACE_A`, `RACE_B`, `RACE_C`, `NOTE` |
| `indoor` | No | boolean | Mark as indoor session |
| `color` | No | string | Calendar color |
| `external_id` | No | string | Your tracking ID (for upsert matching) |

### Output

JSON to stdout. Agent parses this to confirm or report errors.

```json
{
  "success": true,
  "count": 1,
  "events": [
    {"id": 33375903, "name": "Sweet Spot 3x15", "date": "2026-03-05", "type": "Ride", "category": "WORKOUT"}
  ]
}
```

### Validation

push.py validates before sending:
- Name and date required
- Date must be today or future (no retroactive planned workouts)
- Activity type must be valid Intervals.icu type
- Duration capped at 12h, TSS capped at 500 (safety)
- Description syntax checked for basic format errors

If validation fails, nothing is sent. Error returned in JSON.

---

## Intervals.icu Workout Description Syntax

This is the plain-text format Intervals.icu uses for structured workouts. When you push a workout with a `description` field, Intervals.icu parses it into structured steps automatically.

### Basic Step Format

Every step starts with a dash:

```
- [duration OR distance] [target] [optional cadence]
```

### Duration

| Format | Meaning |
|---|---|
| `5m` | 5 minutes |
| `30s` | 30 seconds |
| `1h` | 1 hour |
| `1h30m` | 1 hour 30 minutes |
| `5m30s` | 5 minutes 30 seconds |

**`m` means minutes, not meters.**

### Distance

| Format | Meaning |
|---|---|
| `2km` | 2 kilometers |
| `1mi` | 1 mile |
| `500mtr` | 500 meters |

### Power Targets (Cycling)

| Format | Meaning |
|---|---|
| `75%` | 75% of FTP |
| `88-92%` | Range: 88–92% FTP |
| `220w` | Absolute: 220 watts |
| `200-240w` | Absolute range |
| `Z2` | Power zone 2 |
| `Z3-Z4` | Zone range |

### Heart Rate Targets

| Format | Meaning |
|---|---|
| `70% HR` | 70% of max HR |
| `75-80% HR` | Range of max HR |
| `95% LTHR` | 95% of lactate threshold HR |
| `Z2 HR` | HR zone 2 |

### Pace Targets (Running/Rowing)

| Format | Meaning |
|---|---|
| `60% Pace` | 60% of threshold pace |
| `78-82% Pace` | Pace range |
| `Z2 Pace` | Pace zone 2 |
| `5:00/km Pace` | Absolute pace |

### Cadence

Append `rpm` after the target:

```
- 10m 75% 90rpm
- 5m 88% 70-80rpm
```

### Ramps

Gradual intensity change over a step:

```
- 10m ramp 50%-75%
- 15m ramp 60%-90% 85rpm
```

### Freeride (ERG Off)

```
- 20m freeride
```

### Repeats

Two forms. Always leave a blank line before and after the repeat block:

**Standalone:**
```
3x
- 15m 88-92%
- 5m 55%
```

**With label:**
```
Main Set 3x
- 15m 88-92%
- 5m 55%
```

Nested repeats are **not** supported.

### Text Cues

Text before the first duration becomes the step cue shown to the athlete:

```
- Warmup 15m ramp 50%-65%
- Settle in 5m 65%
```

### Complete Examples

#### Endurance Ride (AE-2 template)

```
- 10m ramp 50%-65%
- 90m 65-75%
- 10m ramp 65%-50%
```

#### Sweet Spot 3×15 (SS-1 template)

```
- 15m ramp 50%-75%

3x
- 15m 88-92%
- 5m 55%

- 10m 50%
```

#### VO2max 5×4 (VO2-1 template)

```
- 15m ramp 50%-75%

5x
- 4m 106-120%
- 3m Z1

- 10m 50%
```

#### Threshold 2×20 (TH-1 template)

```
- 15m ramp 50%-75%

2x
- 20m 95-105%
- 5m 55%

- 10m 50%
```

#### Running Easy (HR-based)

```
- 10m Z1 HR
- 40m Z2 HR
- 5m Z1 HR
```

#### Running Intervals (Pace-based)

```
- 2km 70-75% Pace

4x
- 1km Z4 Pace
- 500mtr Z1 Pace

- 1km 70% Pace
```

### Mapping Section 11 Templates → Description Syntax

The Workout Reference Library (`WORKOUT_REFERENCE.md`) prescribes sessions by template ID (e.g., SS-1, VO2-2). To push these to the calendar, translate the template's zone targets and structure into description syntax:

| Template | Description |
|---|---|
| AE-1 (Short Endurance) | `- 10m ramp 50%-65%\n- 50m 65-75%\n- 10m ramp 65%-50%` |
| AE-2 (Medium Endurance) | `- 10m ramp 50%-65%\n- 90m 65-75%\n- 10m ramp 65%-50%` |
| AE-4 (Recovery) | `- 45m Z1` |
| SS-1 (Sweet Spot 3×15) | `- 15m ramp 50%-75%\n\n3x\n- 15m 88-92%\n- 5m 55%\n\n- 10m 50%` |
| SS-2 (Sweet Spot 2×20) | `- 15m ramp 50%-75%\n\n2x\n- 20m 88-92%\n- 5m 55%\n\n- 10m 50%` |
| TH-1 (Threshold 2×20) | `- 15m ramp 50%-75%\n\n2x\n- 20m 95-105%\n- 5m 55%\n\n- 10m 50%` |
| VO2-1 (VO2max 5×4) | `- 15m ramp 50%-75%\n\n5x\n- 4m 106-120%\n- 3m Z1\n\n- 10m 50%` |
| VO2-2 (VO2max 3×10 short-short) | `- 15m ramp 50%-75%\n\n3x\n- 10m 106-120%\n- 5m Z1\n\n- 10m 50%` |

**Important:** Use `%FTP` ranges from the template, not absolute watts. Intervals.icu resolves `%` to the athlete's current FTP. This means workouts stay correct even if FTP changes between creation and execution.

### What NOT to Do

- **Don't use absolute watts** unless the athlete specifically requests it. `88-92%` adapts to FTP; `250w` doesn't.
- **Don't nest repeats.** Intervals.icu doesn't support it.
- **Don't use `m` for meters.** It means minutes. Use `km`, `mi`, or `mtr`.
- **Don't skip the blank line** around repeat blocks.
- **Don't push past dates.** push.py rejects them. Planned workouts are future-only.
- **Don't push workouts the athlete didn't ask for.** Always get confirmation before writing to their calendar.

---

## OpenClaw / Remote Agents

If the agent runs on a platform that fetches files from GitHub (like OpenClaw), it can pull push.py at runtime:

```
https://raw.githubusercontent.com/CrankAddict/section-11/main/examples/agentic/push.py
```

The agent must have the athlete's Intervals.icu credentials available (typically from DOSSIER.md or workspace config) and network access to `intervals.icu`.

## Dependencies

- Python 3.6+
- `requests` library (same as sync.py)
