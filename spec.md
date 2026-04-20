# spec.md — Obsidian Health Lab Tracking System

## Goal

Create a structured health lab tracking system inside an Obsidian vault that:

- stores laboratory results as individual notes
- aggregates them automatically via Dataview
- supports longitudinal tracking of kidney, liver, and metabolic markers
- is extensible for future biomarker tracking
- works without manual dashboard updates
- contains no personal medical data by default

The system must follow Obsidian + Dataview best practices using YAML frontmatter metadata.

---

# Target Folder Structure

Create the following structure inside the vault root:

Health/
 ├── Dashboard.md
 └── Labs/
      └── _Lab_Template.md

If folders already exist:

- do not overwrite existing files
- only create missing files

---

# Dashboard.md

Create file:

Health/Dashboard.md

Insert content:

# Health Dashboard – Lab Tracking

## Kidney Function – Timeline

```dataview
TABLE date,
creatinine AS "Creatinine",
egfr_creatinine AS "eGFR (Creatinine)",
cystatin_c AS "Cystatin C",
egfr_combined AS "eGFR Combined",
acr_urine AS "ACR"
FROM "Health/Labs"
WHERE type = "lab"
SORT date ASC
```

---

## Liver Function – Timeline

```dataview
TABLE date,
ggt AS "GGT",
ast AS "AST",
alt AS "ALT",
ultrasound_liver AS "Ultrasound Liver"
FROM "Health/Labs"
WHERE type = "lab"
SORT date ASC
```

---

## Electrolytes – Timeline

```dataview
TABLE date,
sodium AS "Na",
potassium AS "K",
calcium AS "Ca"
FROM "Health/Labs"
WHERE type = "lab"
SORT date ASC
```

---

## Measurement Context

```dataview
TABLE date,
training_48h_before AS "Training",
creatine_supplementation AS "Creatine",
hydration_status AS "Hydration"
FROM "Health/Labs"
WHERE type = "lab"
SORT date ASC
```

---

# Lab Template File

Create file:

Health/Labs/_Lab_Template.md

Insert content:

---
type: lab
date:

creatinine:
egfr_creatinine:
cystatin_c:
egfr_combined:
acr_urine:

ggt:
ast:
alt:

sodium:
potassium:
calcium:

ultrasound_liver:
ultrasound_kidney:

training_48h_before:
creatine_supplementation:
hydration_status:

notes:
---

# Lab Result

## Kidney Function

Creatinine:
eGFR (Creatinine):
Cystatin C:
eGFR Combined:
ACR Urine:

---

## Liver Function

GGT:
AST:
ALT:

Ultrasound Liver:
Ultrasound Kidney:

---

## Context

Training 48h before test:
Creatine supplementation:
Hydration:

---

## Interpretation

Free text interpretation

---

# Requirements

Implementation must ensure:

1. YAML frontmatter syntax remains valid
2. folder creation is recursive
3. existing files are not overwritten
4. markdown formatting preserved exactly
5. dataview code blocks unchanged
6. compatible with default Dataview plugin configuration
7. ISO date format (YYYY-MM-DD)
8. no personal data included by default

---

# Optional Future Extension Hooks

Reserve metadata fields for future tracking:

hba1c
triglycerides
hdl
ldl
fasting_insulin
body_weight
waist_circumference
vitamin_d

These fields should remain compatible with future Dataview queries.

---

# Success Criteria

System is correctly installed if:

- Dashboard.md renders Dataview tables
- Labs folder exists
- template file exists
- new lab entries appear automatically in dashboard tables
- no manual linking required
