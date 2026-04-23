---
description: Extract a doctor's visit or imaging report from one or more files and create an Obsidian visit note
allowed-tools: [Read, Write, Bash]
---

Import a doctor's visit summary or imaging report from one or more files into a single Obsidian visit note.

Files to import: $ARGUMENTS

## Steps

### 1. Resolve the files

Parse `$ARGUMENTS` as a space-separated list of file paths. Quoted paths (with spaces) are respected.
If `$ARGUMENTS` is empty, ask the user: "Please provide one or more paths to visit letters, imaging reports, or photos."

For each file:
- If it is a `.heic` file, convert it first:
  ```bash
  sips -s format jpeg -r 90 --resampleWidth 1600 "{input}" --out /tmp/visit_import_{n}.jpg
  ```
  Then read the converted file.
- If it is a PDF, use `pdftoppm` if needed (already shown to work in this project).
- Otherwise read directly.

### 2. Read and extract

Read each file in sequence using the Read tool. Merge the extracted fields into a single result — later files supplement earlier ones (e.g. a separate imaging appendix adds to the main letter). If the same field appears in multiple files with conflicting values, flag it in the summary (step 3) and ask the user to choose.

Extract these fields:

| Field | German label | Notes |
|---|---|---|
| `date` | Untersuchungsdatum / Befunddatum | YYYY-MM-DD |
| `visit_type` | — | Derive: imaging report → `imaging`; specialist letter → `specialist`; GP → `consultation`; follow-up → `follow-up` |
| `specialty` | Fachrichtung / Arztbezeichnung | e.g. Radiologie, Orthopädie |
| `doctor` | Arzt / Unterzeichner | Name of signing doctor |
| `facility` | Praxis / Klinik | Name of practice or hospital |
| `referring_doctor` | Überweiser / Zuweiser | Referring doctor if mentioned |
| `reason` | Indikation / Überweisungsgrund | Presenting complaint or indication |
| `findings` | Befund | Full findings text |
| `assessment` | Beurteilung | Summary / conclusion |
| `diagnoses` | Diagnose / ICD | Diagnoses listed |
| `medications_changed` | Medikation | New, changed, or stopped medications |
| `referrals` | Weiterüberweisung | Further referrals mentioned |
| `next_appointment` | Wiedervorstellung / Kontrolltermin | Follow-up date if mentioned |
| `modality` | Modalität | For imaging: MRT / CT / Röntgen / Ultraschall |
| `body_part` | Körperregion | For imaging: body region examined |

Rules:
- Convert `date` to YYYY-MM-DD
- Leave any undetected field empty
- **YAML values must be safe strings:** always wrap `reason`, `diagnoses`, `assessment`, `medications_changed`, `referrals`, and `next_appointment` in double quotes. Keep them short (one-line summaries) — no colons, no em dashes (`–`), no unescaped special characters. Full text goes in the markdown body only.
- For `findings` and `assessment` in the **body**: copy the text faithfully, preserving medical terminology

### 3. Show summary and confirm

Display extracted fields in a table. Then ask:
> "Does this look correct? I'll create `Health/Visits/{date}-{slug}.md` — proceed?"

Where `{slug}` is a short kebab-case label derived from specialty + body_part or reason (e.g. `mrt-handgelenk`, `orthopaedie-kontrolle`).

Wait for confirmation. Apply any corrections the user provides.

### 4. Write the note

Vault path: `/Users/mswientek/Library/Mobile Documents/iCloud~md~obsidian/Documents/pkm-vault`

Create the file at:
```
{vault_path}/Health/Visits/{date}-{slug}.md
```

If the file already exists, **stop and report** — do not overwrite.

Use this exact structure:

```
---
title: {specialty / visit_type} – {date}
type: visit
status: draft
source: manual
created: {today in YYYY-MM-DD}
date: {date}

visit_type: {value or leave empty}
specialty: {value or leave empty}
doctor: {value or leave empty}
facility: {value or leave empty}
referring_doctor: {value or leave empty}

reason: {value or leave empty}
diagnoses: {value or leave empty}
assessment: {value or leave empty}
medications_changed: {value or leave empty}
referrals: {value or leave empty}
next_appointment: {value or leave empty}

modality: {value or leave empty}
body_part: {value or leave empty}
---

# {specialty / visit_type} – {date}

**Doctor:** {doctor} ({specialty})
**Facility:** {facility}
**Referring doctor:** {referring_doctor}
**Type:** {visit_type}

---

## Reason / Indication

{reason}

---

## Findings

{findings — full text}

---

## Assessment

{assessment — full text}

---

## Diagnoses

{diagnoses, one per bullet, or —}

---

## Medications Changed

{medications_changed, or —}

---

## Referrals

{referrals, or —}

---

## Next Steps

- [ ]

---

## Next Appointment

{next_appointment or —}
```

### 5. Report

Print the full path of the created file. Done.

---

## Installation

```bash
cp skills/visit-import.md ~/.claude/commands/
```

Then use as:
```
/visit-import /path/to/arztbrief.pdf
/visit-import /path/to/letter.pdf /path/to/imaging-appendix.pdf
/visit-import "/path with spaces/report.heic" /path/to/page2.jpg
```
