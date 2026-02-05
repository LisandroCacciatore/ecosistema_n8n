 Training Data Model - Strength Programming Database

> **Objective**: Standardize and compare different strength training methodologies (Soviet, Bulgarian, Westside, 5/3/1, etc.) through a unified data model that enables data-driven analysis.

## Problem Statement

Different strength programs use different terminology, periodization schemes, and metrics. Comparing effectiveness across systems requires normalizing these into a common schema where **data can be compared with data**.

This repository defines a **canonical data model** for ingesting, validating, and analyzing strength training sessions from heterogeneous programming methodologies.

---

## Table of Contents

- [Core Taxonomy](#core-taxonomy)
- [Database Schema](#database-schema)
- [System Mappings](#system-mappings)
- [Use Cases](#use-cases)
- [Implementation](#implementation)

---

## Core Taxonomy

Every training session, regardless of methodology, can be decomposed into these fundamental concepts:

| Concept | Definition | Example |
|---------|------------|---------|
| **Program** | Training methodology/system | "Westside Conjugate", "5/3/1" |
| **Cycle** | Temporal block (macro/meso/micro) | "Hypertrophy Mesocycle Week 1-4" |
| **Session** | Single training day | "Monday ME Lower 2026-02-03" |
| **Event** | Work block for ONE exercise | "Back Squat 5x5 @ 75%" |
| **Set** | Individual set within event | "Set 3: 5 reps @ 150kg, RPE 8" |
| **Stimulus** | Physiological adaptation target | MaxStrength, Speed, Hypertrophy |
| **Dose** | Training load quantification | Volume (sets×reps×load), density, rest |
| **Response** | Athlete feedback | RPE, soreness, HRV, sleep quality |

### Stimulus Categories (Controlled Vocabulary)

```
- MaxStrength       // Singles/doubles @ 90%+
- Speed_Power       // <60% with velocity intent
- Hypertrophy       // 6-12 reps, moderate load
- WorkCapacity      // High volume, short rest
- Technique         // Skill acquisition, low fatigue
- Recovery          // Active recovery, mobility
```

---

## Database Schema

### Table: `programs`
```sql
CREATE TABLE programs (
  program_id STRING PRIMARY KEY,
  name STRING NOT NULL,              -- "Westside Conjugate"
  author STRING,                     -- "Louie Simmons"
  description STRING,
  typical_length_weeks INT
);
```

### Table: `cycles`
```sql
CREATE TABLE cycles (
  cycle_id STRING PRIMARY KEY,
  program_id STRING REFERENCES programs,
  type STRING,                       -- 'macrocycle' | 'mesocycle' | 'microcycle'
  start_date DATE,
  end_date DATE,
  phase_label STRING                 -- 'Base' | 'Intensification' | 'Peaking'
);
```

### Table: `sessions`
```sql
CREATE TABLE sessions (
  session_id STRING PRIMARY KEY,
  cycle_id STRING REFERENCES cycles,
  athlete_id STRING,
  date DATE NOT NULL,
  session_type STRING,               -- 'ME', 'DE', 'Accessory', 'Heavy', 'Volume'
  objective STRING,
  stimulus_primary STRING            -- From controlled vocabulary
);
```

### Table: `events`
```sql
CREATE TABLE events (
  event_id STRING PRIMARY KEY,
  session_id STRING REFERENCES sessions,
  exercise_id STRING REFERENCES exercises,
  stimulus_primary STRING NOT NULL,
  sets_planned INT,
  reps_scheme_text STRING,           -- "5x5" or "1-1-1-1-1"
  load_pct FLOAT,                    -- % of 1RM
  load_absolute FLOAT,               -- Actual kg/lbs
  tempo STRING,                      -- "2020" (eccentric-pause-concentric-pause)
  rest_seconds INT,
  notes STRING
);
```

### Table: `sets`
```sql
CREATE TABLE sets (
  set_id STRING PRIMARY KEY,
  event_id STRING REFERENCES events,
  set_number INT,
  reps INT,
  load_kg FLOAT,
  actual_rpe FLOAT,                  -- 1-10 scale
  fail_flag BOOLEAN                  -- Did the set fail?
);
```

### Table: `exercises`
```sql
CREATE TABLE exercises (
  exercise_id STRING PRIMARY KEY,
  name STRING NOT NULL,              -- "Back Squat"
  variation STRING,                  -- "Competition", "High Bar", "Box"
  primary_muscle STRING,
  category STRING                    -- 'Main Lift' | 'Accessory'
);
```

### Table: `responses`
```sql
CREATE TABLE responses (
  response_id STRING PRIMARY KEY,
  session_id STRING REFERENCES sessions,
  athlete_id STRING,
  date_time TIMESTAMP,
  rpe_post FLOAT,                    -- Session RPE
  soreness INT,                      -- 1-10 scale
  hr_rest INT,                       -- Resting heart rate
  sleep_quality INT                  -- 1-10 scale
);
```

---

## System Mappings

How different methodologies map to the common schema:

### Soviet System (Base Volume → Intensification)

**Session Example**: Base Phase Squat
```json
{
  "session_type": "Base Volume",
  "stimulus_primary": "Strength_Hypertrophy",
  "events": [
    {
      "exercise": "Back Squat - Competition",
      "sets_planned": 5,
      "reps_scheme_text": "5x5",
      "load_pct": 72,
      "tempo": "2020",
      "rest_seconds": 180
    }
  ]
}
```

**Characteristics**:
- Medium-high volume (5x5, 6x4)
- 70-85% intensity
- Gradual progression over weeks
- RPE typically 6-8

---

### Bulgarian Method (Abadjiev - High Frequency Maximals)

**Session Example**: Daily Max C&J
```json
{
  "session_type": "Competition Lift Max",
  "stimulus_primary": "MaxStrength",
  "session_frequency": 2,           // Sessions per day
  "events": [
    {
      "exercise": "Clean & Jerk",
      "sets_planned": 6,
      "reps_scheme_text": "6x1 to max",
      "load_pct": [90, 93, 96, 98, 101, 102],
      "rest_seconds": 180
    }
  ]
}
```

**Characteristics**:
- 1-3 sessions/day
- Singles @ 90-105%
- Minimal assistance work
- RPE 9-10 on max attempts
- **Critical metadata**: `session_time_of_day`, `session_frequency`

---

### Westside Conjugate (Simmons - Max/Dynamic Rotation)

**Session Example**: Dynamic Effort Lower
```json
{
  "session_type": "DE",
  "stimulus_primary": "Speed_Power",
  "events": [
    {
      "exercise": "Box Squat",
      "sets_planned": 8,
      "reps_scheme_text": "8x3",
      "load_pct": 55,
      "tempo": "explosive",
      "rest_seconds": 60
    },
    {
      "exercise": "Romanian Deadlift",
      "sets_planned": 4,
      "reps_scheme_text": "4x8",
      "load_pct": 60,
      "rest_seconds": 90
    }
  ]
}
```

**Characteristics**:
- ME days: 1-3 reps @ 90%+
- DE days: 8-12 sets × 2-3 reps @ 50-60%
- Rotating exercises every 1-3 weeks
- Tags: `ME-day`, `DE-day`, `RE-day`

---

### 5/3/1 (Wendler - Linear % Progression)

**Session Example**: Week 3 Squat
```json
{
  "session_type": "Main Lift",
  "stimulus_primary": "MaxStrength",
  "phase_label": "Week 3 (5/3/1+)",
  "events": [
    {
      "exercise": "Back Squat",
      "sets": [
        {"set_number": 1, "reps": 5, "load_pct": 75},
        {"set_number": 2, "reps": 3, "load_pct": 85},
        {"set_number": 3, "reps": "AMRAP", "load_pct": 95}
      ]
    }
  ]
}
```

**Characteristics**:
- Fixed % progression per week
- AMRAP final set (captures actual reps for volume)
- Easy athlete-to-athlete comparison via `load_pct`
- Autoregulation through AMRAP

---

### Cube Method (Lilly - Rotating Priorities)

**Session Example**: Heavy Week Squat
```json
{
  "session_type": "Heavy",
  "rotation_phase": "Heavy",         // Rotates: Heavy → Volume → Speed
  "stimulus_primary": "MaxStrength",
  "events": [
    {
      "exercise": "Back Squat",
      "sets_planned": 2,
      "reps_scheme_text": "2x1 @ 95%+",
      "load_pct": 97
    },
    {
      "exercise": "Back Squat",
      "sets_planned": 3,
      "reps_scheme_text": "3x5 backoff",
      "load_pct": 70
    }
  ]
}
```

**Characteristics**:
- 3-week rotation: Heavy → Volume → Speed
- Each week prioritizes different adaptation
- Cycle attribute: `rotation = ['Heavy', 'Volume', 'Speed']`

---

## Use Cases

What questions can this database answer?

### 1. Volume Load Comparison
```sql
-- Total volume per stimulus across programs
SELECT 
  p.name,
  e.stimulus_primary,
  SUM(s.reps * s.load_kg * e.sets_planned) as total_volume_kg
FROM events e
JOIN sessions ses ON e.session_id = ses.session_id
JOIN cycles c ON ses.cycle_id = c.cycle_id
JOIN programs p ON c.program_id = p.program_id
JOIN sets s ON e.event_id = s.event_id
WHERE c.phase_label = 'Base'
GROUP BY p.name, e.stimulus_primary;
```

### 2. Frequency Tolerance Analysis
```sql
-- Average RPE by training frequency
SELECT 
  athlete_id,
  COUNT(DISTINCT DATE(date)) as training_days_per_week,
  AVG(rpe_post) as avg_session_rpe
FROM sessions s
JOIN responses r ON s.session_id = r.session_id
WHERE date BETWEEN '2026-01-01' AND '2026-01-31'
GROUP BY athlete_id;
```

### 3. Program Effectiveness
```sql
-- 1RM improvement by program over 12 weeks
SELECT 
  p.name,
  athlete_id,
  MAX(load_kg) - MIN(load_kg) as rm_improvement
FROM events e
JOIN sessions s ON e.session_id = s.session_id
JOIN cycles c ON s.cycle_id = c.cycle_id
JOIN programs p ON c.program_id = p.program_id
WHERE e.reps_scheme_text LIKE '%1RM%'
  AND c.type = 'mesocycle'
GROUP BY p.name, athlete_id;
```

---

## Implementation

### Design Principles

1. **Standardize stimulus categories** - Use controlled vocabulary (enum)
2. **Store both planned & actual** - `planned_load_pct` vs `actual_load_kg`
3. **Capture frequency metadata** - Day, time, session # per day
4. **Phase labels are mandatory** - Every cycle must have `phase_label`
5. **Store percentages AND absolute values** - `load_pct` + `load_kg`

### Data Ingestion Rules

When ingesting from Google Sheets or other sources:

```
✅ DO:
- Normalize exercise names ("Back Squat" not "backsquat" or "BS")
- Validate RPE is 1-10
- Require phase_label for every session
- Store original source in metadata (provenance)

❌ DON'T:
- Allow free-text stimulus values
- Skip tempo/rest data (critical for analysis)
- Lose planned vs actual distinction
```

### Next Steps

1. **Define validation schema** - JSON Schema for event/session structure
2. **Build n8n ingestion workflow** - Implement Workflow #1 (Gatekeeper)
3. **Create derived tables** - Weekly aggregations, rolling averages
4. **Build comparison dashboards** - Looker Studio / Tableau views

---

## Contributing

This is an open data model. If you implement additional training systems (Conjugate Periodization, Block Periodization, DUP, etc.), submit a PR with:

1. System mapping example (JSON)
2. Key characteristics
3. Critical metadata fields

---
