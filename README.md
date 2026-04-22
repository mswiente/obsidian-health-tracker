# obsidian-health-tracker

Health lab tracking system inside an Obsidian vault. Stores blood and urine lab results as individual notes and aggregates them automatically via Dataview — no manual dashboard updates needed.

## Features

- Longitudinal tracking of kidney, liver, and metabolic markers
- Automatic aggregation via Dataview queries (no manual linking)
- Measurement context tracking (training, creatine supplementation, hydration)
- `/lab-import` Claude Code skill: extract values from a lab result screenshot or PDF and create the Obsidian note automatically
- Extensible for future biomarkers (HbA1c, lipids, vitamin D, etc.)

## Vault Structure

```
Health/
├── Dashboard.md          # Dataview dashboard — auto-populated from lab notes
└── Labs/
    ├── _Lab_Template.md  # Template for new lab entries (used by Obsidian's Template plugin)
    └── YYYY-MM-DD-lab-results.md  # Individual lab notes (one per visit)
```

## Installation

### Prerequisites

- [Obsidian](https://obsidian.md) with the [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) installed and enabled
- An existing Obsidian vault

### 1. Create the vault files

Copy `Health/Dashboard.md` and `Health/Labs/_Lab_Template.md` into your vault, preserving the folder structure. Or run Claude Code in this repo — the files were written directly to the vault during setup.

The `Health/` folder must live at the vault root for the Dataview queries to resolve correctly.

### 2. Install the `/lab-import` skill (optional)

The skill lets you point Claude Code at a lab result image or PDF and have it extract values and create the Obsidian note automatically.

```bash
cp skills/lab-import.md ~/.claude/commands/
```

Restart Claude Code, then use it as:

```
/lab-import /path/to/lab-screenshot.png
/lab-import /path/to/blutbefund.pdf
```

The skill handles both German and English lab report field names.

## Usage

### Adding a lab result manually

1. In Obsidian, create a new note in `Health/Labs/` using `_Lab_Template.md` as the template
2. Name it `YYYY-MM-DD-lab-results.md` (e.g. `2026-03-15-lab-results.md`)
3. Fill in the YAML frontmatter fields with your values
4. Open `Health/Dashboard.md` — the new entry appears automatically in all four tables

### Adding a lab result via `/lab-import`

1. Run `/lab-import /path/to/your-lab-report.png` in Claude Code
2. Claude reads the image/PDF and extracts available values
3. Review the extraction summary and confirm (or correct values)
4. Claude writes the note to `Health/Labs/`
5. Open Obsidian — the note appears in the dashboard immediately

### Dashboard tables

| Table | Fields |
|---|---|
| Kidney Function | Creatinine, eGFR (Creatinine), Cystatin C, eGFR Combined, ACR |
| Liver Function | GGT, AST, ALT, Ultrasound Liver |
| Electrolytes | Na, K, Ca |
| Measurement Context | Training 48h before, Creatine supplementation, Hydration |

## Lab Note Frontmatter

Each lab note uses these YAML fields:

```yaml
---
title: Lab Results YYYY-MM-DD
type: lab
status: draft
source: manual
created: YYYY-MM-DD
date: YYYY-MM-DD        # used by Dataview for sorting and filtering

# Kidney
creatinine:             # numeric only, no units
egfr_creatinine:
cystatin_c:
egfr_combined:
acr_urine:

# Liver
ggt:
ast:
alt:

# Electrolytes
sodium:
potassium:
calcium:

# Imaging
ultrasound_liver:
ultrasound_kidney:

# Measurement context
training_48h_before:    # yes / no
creatine_supplementation: # yes / no
hydration_status:       # normal / low / high

notes:
---
```

## Future Extensions

### Additional lab markers

The following fields are reserved for future Dataview queries and can be added to any lab note:

```yaml
hba1c:
triglycerides:
hdl:
ldl:
fasting_insulin:
body_weight:
waist_circumference:
vitamin_d:
```

Additional panels to add from previous tests:

- **Thyroid:** `tsh`, `ft3`, `ft4`
- **Iron / Anaemia:** `ferritin`, `iron`, `transferrin_saturation`, `hemoglobin`
- **Inflammation:** `crp`, `leukocytes`
- **Vitamins:** `vitamin_b12`, `folate`, `vitamin_d` (already reserved)

Each new panel needs a corresponding Dataview table added to `Health/Dashboard.md`.

### Doctor's visit notes

Add a `Health/Visits/` folder with a `_Visit_Template.md` for GP and specialist appointments. Fields to track:

- `type: visit`
- `date`, `doctor`, `specialty`
- `reason` — presenting complaint
- `findings` — examination / test results discussed
- `diagnoses`
- `medications_changed` — new / stopped / adjusted
- `referrals`
- `next_appointment`

Also add a `/visit-import` skill (parallel to `/lab-import`) that extracts structured data from visit summary letters or photos of doctor's notes.

### Dashboard enhancements

Extend `Health/Dashboard.md` with:

- **Recent Visits** — Dataview table from `Health/Visits/` showing last 5 entries (date, doctor, reason, next appointment)
- **Current Issues** — inline list or Dataview query of open diagnoses / active problems
- **Next Steps** — Dataview task list (`- [ ]`) aggregated across visit and lab notes for upcoming appointments, referrals, and follow-up tests

## PKM Integration

This system is designed to integrate with [pkm.ai](https://github.com/mswientek/pkm.ai). A `pkm lab create` command (deferred) will create lab notes from the CLI without opening Obsidian.
