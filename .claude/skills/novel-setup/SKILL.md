---
name: novel-setup
description: Initialize a new novel project by creating the wiki directory structure (characters, plot, world, themes). Run this once before using /novel-write.
---

# Novel Setup

Set up the story wiki folder for a new novel project.

## Steps

### 1. Gather project details

Ask the user (one message, all at once):
- **Novel title** — working title for this project
- **Genre** — e.g. "epic fantasy", "noir thriller", "literary fiction", "science fiction"
- **Premise** — one or two sentences describing the core story idea
- **Wiki directory** — where to store story context (default: `./wiki/`)

### 2. Create the wiki directory

Use Bash to create the directory:
```powershell
New-Item -ItemType Directory -Force -Path "<wiki_dir>"
```

### 3. Create `project.md`

Write a file at `<wiki_dir>/project.md` with:
```markdown
# [NOVEL_TITLE]

**Genre:** [GENRE]

**Premise:** [PREMISE]
```

### 4. Create the four wiki files

Create each with a template header. Only create files that do not already exist.

**`<wiki_dir>/characters.md`:**
```markdown
# Characters

<!-- Add character entries below. -->
<!-- Format: **Name** — description, key traits, role in story -->
```

**`<wiki_dir>/plot.md`:**
```markdown
# Plot Events

<!-- Add plot events in chronological order. -->
<!-- Format: **Ch. N / Scene** — what happened, consequences -->
```

**`<wiki_dir>/world.md`:**
```markdown
# World Facts

<!-- Add world-building facts: geography, history, magic/tech rules, factions, customs -->
```

**`<wiki_dir>/themes.md`:**
```markdown
# Themes

<!-- Add recurring themes, motifs, and symbolic elements -->
```

### 5. Confirm to user

Print:
```
Novel project initialized.

Wiki: <wiki_dir>/
  ├── project.md      ← title, genre, premise
  ├── characters.md   ← character roster
  ├── plot.md         ← plot events log
  ├── world.md        ← world-building facts
  └── themes.md       ← themes & motifs

Next: use /novel-write to generate a chapter.
```

## Notes

- If any wiki file already exists, do NOT overwrite it — skip it and tell the user.
- The wiki directory is used by `/novel-write` and `/novel-wiki`. Keep it in the project root or pass `--wiki <path>` to those skills.
