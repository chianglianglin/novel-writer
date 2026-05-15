---
name: novel-wiki
description: Browse and edit the story wiki (characters, plot, world, themes). View current state, add entries, or search for specific facts.
---

# Novel Wiki

Browse, search, and edit the story wiki for your novel project.

## Step 1: Determine wiki directory

Check if `--wiki <path>` was passed as an argument. If not, default to `./wiki/`.

## Step 2: Read all wiki files

Read the following files using the Read tool:
- `<wiki_dir>/project.md` — title, genre, premise
- `<wiki_dir>/characters.md` — character roster
- `<wiki_dir>/plot.md` — plot events log
- `<wiki_dir>/world.md` — world-building facts
- `<wiki_dir>/themes.md` — themes and motifs

If a file does not exist, note it as `[file not found]`.

## Step 3: Display the wiki

Print the wiki contents in a clean, readable format:

```
═══════════════════════════════════════════
  STORY WIKI
═══════════════════════════════════════════

PROJECT
-------
[project.md content]

CHARACTERS
----------
[characters.md content]

PLOT EVENTS
-----------
[plot.md content]

WORLD FACTS
-----------
[world.md content]

THEMES
------
[themes.md content]

═══════════════════════════════════════════
```

## Step 4: Offer actions

Ask the user what they'd like to do:

1. **Add an entry** — add a new character, plot event, world fact, or theme
2. **Edit an entry** — modify an existing entry
3. **Search** — find entries matching a keyword
4. **Done** — exit

### If "Add an entry":

Ask:
- Which section? (characters / plot / world / themes)
- What is the entry text?

Append to the appropriate file. Format:
```markdown
- **[entry text]**
```

Or if the user wants to add under a specific chapter heading, prompt for the chapter number and append:
```markdown

## After Chapter [N]

- [entry text]
```

Confirm: `Added to <section>. Updated <wiki_dir>/<file>.`

### If "Edit an entry":

Ask which section and what the existing text looks like (partial match is fine).
Read the file, find the matching line, ask the user what it should be changed to.
Use the Edit tool to make the change.
Confirm: `Updated entry in <section>.`

### If "Search":

Ask for a keyword.
Search all wiki files for lines containing the keyword (case-insensitive).
Display matching lines with their file source and line number.

### If "Done":

Exit the skill.

## Notes

- Run `/novel-setup` first if the wiki directory does not exist.
- The wiki is plain markdown — you can also edit the files directly in any text editor.
- `/novel-write` automatically appends new facts to the wiki after each chapter.
