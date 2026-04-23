---
description: Extract a doctor's visit or imaging report and create an Obsidian visit note
allowed-tools: [Read, Write, Bash]
---

Import a doctor's visit summary or imaging report into an Obsidian visit note.

File to import: $ARGUMENTS

## Steps

### 1. Resolve the file

If `$ARGUMENTS` is empty, ask the user: "Please provide the path to the visit letter, imaging report, or photo."
Otherwise use the provided path directly. If it is a HEIC file, convert it first:
```bash
sips -s format jpeg -r 90 --resampleWidth 1600 "{input}" --out /tmp/visit_import.jpg
```
Then read `/tmp/visit_import.jpg`.

### 2. Read and extract

Use the Read tool to open the file. Extract these fields:

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
- For `findings` and `assessment`: copy the text faithfully, preserving medical terminology

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

Then use as: `/visit-import /path/to/report.pdf`
