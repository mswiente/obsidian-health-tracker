---
description: Extract lab results from one or more images/PDFs and create an Obsidian lab note
allowed-tools: [Read, Write, Bash]
---

Import lab results from one or more images or PDFs into a single Obsidian lab note.

Files to import: $ARGUMENTS

## Steps

### 1. Resolve the files

Parse `$ARGUMENTS` as a space-separated list of file paths. Quoted paths (with spaces) are respected.
If `$ARGUMENTS` is empty, ask the user: "Please provide one or more paths to lab result images or PDFs."

For each file:
- If it is a `.heic` file, convert it first:
  ```bash
  sips -s format jpeg --resampleWidth 1600 "{input}" --out /tmp/lab_import_{n}.jpg
  ```
  Then read the converted file.
- If it is a PDF, use `pdftoppm` if needed (already shown to work in this project).
- Otherwise read directly.

### 2. Read and extract

Read each file in sequence using the Read tool. Merge the extracted values into a single result set — later files supplement earlier ones. If the same field appears in multiple files with different values, flag the conflict in the summary (step 4) and ask the user to choose.

Extract all available values and map German or English field names to these YAML keys:

| YAML key | German label | English label |
|---|---|---|
| `date` | Entnahmedatum / Befunddatum | Collection date |
| `creatinine` | Kreatinin | Creatinine |
| `egfr_creatinine` | eGFR (Kreatinin) / CKD-EPI Kreatinin | eGFR (Creatinine) |
| `cystatin_c` | Cystatin C | Cystatin C |
| `egfr_combined` | eGFR kombiniert / CKD-EPI Cystatin+Krea | eGFR Combined |
| `acr_urine` | Albumin-Kreatinin-Quotient / ACR Urin | ACR Urine |
| `ggt` | Gamma-GT / GGT | GGT |
| `ast` | AST / GOT | AST |
| `alt` | ALT / GPT | ALT |
| `ultrasound_liver` | Sono Leber / Leberultraschall | Ultrasound Liver |
| `ultrasound_kidney` | Sono Niere / Nierenultraschall | Ultrasound Kidney |
| `sodium` | Natrium / Na | Sodium |
| `potassium` | Kalium / K | Potassium |
| `calcium` | Kalzium / Ca | Calcium |

Rules:
- Store **numeric values only** in YAML (no units, no reference ranges)
- Convert `date` to `YYYY-MM-DD` format
- For descriptive fields (`ultrasound_liver`, `ultrasound_kidney`): store a short quoted string (e.g. `"unauffällig"`, `"normal"`)
- Leave any undetected field empty (just `field:` with no value) — never guess or interpolate
- **YAML string values must be safe:** wrap any text value in double quotes; avoid colons, em dashes (`–`), and unescaped special characters inside YAML values

### 3. Ask for context fields

If these are not present on the report, ask the user:
- "Training in the 48h before the test? (yes/no)"
- "Creatine supplementation active? (yes/no)"
- "Hydration status? (normal/low/high)"

### 4. Show summary and confirm

Display extracted values in a markdown table (found vs. empty). Then ask:
> "Does this look correct? I'll create `Health/Labs/{date}-lab-results.md` — proceed? You can correct any values before I write the file."

Wait for confirmation. Apply any corrections the user provides.

### 5. Write the note

Vault path: `/Users/mswientek/Library/Mobile Documents/iCloud~md~obsidian/Documents/pkm-vault`

Create the file at:
```
{vault_path}/Health/Labs/{date}-lab-results.md
```

If the file already exists, **stop and report** — do not overwrite.

Use this exact structure:

```
---
title: Lab Results {date}
type: lab
status: draft
source: manual
created: {today in YYYY-MM-DD}
date: {date}

creatinine: {value or leave empty}
egfr_creatinine: {value or leave empty}
cystatin_c: {value or leave empty}
egfr_combined: {value or leave empty}
acr_urine: {value or leave empty}

ggt: {value or leave empty}
ast: {value or leave empty}
alt: {value or leave empty}

sodium: {value or leave empty}
potassium: {value or leave empty}
calcium: {value or leave empty}

ultrasound_liver: {value or leave empty}
ultrasound_kidney: {value or leave empty}

training_48h_before: {value or leave empty}
creatine_supplementation: {value or leave empty}
hydration_status: {value or leave empty}

notes:
---

# Lab Results {date}

| Marker | Value | Unit | Reference | Status |
|---|---|---|---|---|
| Creatinine | {value or —} | {unit} | {ref range} | {↑ / ↓ / ✓ or —} |
| eGFR (Creatinine) | {value or —} | ml/min/1.73m² | >90 | {↑ / ↓ / ✓ or —} |
| Cystatin C | {value or —} | {unit} | {ref range} | {↑ / ↓ / ✓ or —} |
| eGFR Combined | {value or —} | ml/min/1.73m² | {ref range} | {↑ / ↓ / ✓ or —} |
| ACR Urine | {value or —} | {unit} | {ref range} | {↑ / ↓ / ✓ or —} |
| GGT | {value or —} | U/l | {ref range} | {↑ / ↓ / ✓ or —} |
| AST | {value or —} | U/l | {ref range} | {↑ / ↓ / ✓ or —} |
| ALT | {value or —} | U/l | {ref range} | {↑ / ↓ / ✓ or —} |
| Sodium | {value or —} | mmol/l | {ref range} | {↑ / ↓ / ✓ or —} |
| Potassium | {value or —} | mmol/l | {ref range} | {↑ / ↓ / ✓ or —} |
| Calcium | {value or —} | mmol/l | {ref range} | {↑ / ↓ / ✓ or —} |

Include reference ranges as printed on the report. Use ✓ for within range, ↑/↓ for flagged, — for not measured.

---

## Kidney Function

Creatinine: {value with unit, e.g. "85 µmol/L"}
eGFR (Creatinine): {value, e.g. "78 mL/min/1.73m²"}
Cystatin C: {value}
eGFR Combined: {value}
ACR Urine: {value}

---

## Liver Function

GGT: {value}
AST: {value}
ALT: {value}

Ultrasound Liver: {value}
Ultrasound Kidney: {value}

---

## Context

Training 48h before test: {value}
Creatine supplementation: {value}
Hydration: {value}

---

## Interpretation

{Write a brief factual summary of notable findings. For each value outside the reference range, note the value, the direction (↑/↓), and any plausible context factor (e.g. training, supplementation, hydration). End with a one-liner on everything that was within range. Do not give medical advice.}

```

### 6. Report

Print the full path of the created file. Done.

---

## Installation

```bash
cp skills/lab-import.md ~/.claude/commands/
```

Then use as:
```
/lab-import /path/to/lab-report.png
/lab-import /path/to/page1.pdf /path/to/page2.pdf
/lab-import "/path with spaces/report.heic" /path/to/page2.jpg
```
