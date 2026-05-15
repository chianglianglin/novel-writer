---
name: novel-review
description: Run a four-pass literary review of a chapter — produces a scored editorial report with specific improvement directives, and optionally applies rewrites and saves an improved version.
---

# Novel Review

Run a four-pass literary review of a chapter, produce a scored editorial report with specific improvement directives, then optionally apply rewrites and save an improved version.

## Step 1: Load context

Accept `--chapter N` or a file path. If no argument, use the most recent chapter in `./output/`.

Read:
- The target chapter file
- `./wiki/characters.md`
- `./wiki/plot.md`
- `./wiki/world.md`
- `./wiki/themes.md`
- The previous chapter file if it exists (for continuity)

## Step 2: Four-pass review

Run each pass independently. For each pass, produce:
- A score 0–100
- A list of specific findings with line-level references where possible

**Pass 1 — Voice & Prose**
Evaluate: sentence rhythm and variety, word-level precision, paragraph transitions, overused words or phrases, moments where the prose is especially strong or weak.

**Pass 2 — Narrative Coherence**
Evaluate: does the chapter achieve the goals stated in its brief (the chapter header)? Do characters behave consistently with their wiki entries? Are plot events internally consistent? Does the emotional arc land?

**Pass 3 — World-building Consistency**
Evaluate: do all details (physical, cultural, magical) match `./wiki/world.md`? Any contradictions, anachronisms, or facts introduced without wiki backing?

**Pass 4 — Pacing & Structure**
Evaluate: are scene beats well-proportioned? Does the chapter open strongly and close with impact? Does it sag in the middle? Is the chapter-level emotional journey complete?

## Step 3: Editorial report

Print this report in full:

```
╔══════════════════════════════════════════╗
  Novel Review — Chapter [N]
╠══════════════════════════════════════════╣
  Voice & Prose:         [score]/100
  Narrative Coherence:   [score]/100
  World Consistency:     [score]/100
  Pacing & Structure:    [score]/100
  Overall:               [average]/100
╠══════════════════════════════════════════╣
  CRITICAL — must fix:
    • [specific issue with location/line]
    • ...

  IMPORTANT — should fix:
    • [specific issue]
    • ...

  MINOR — nice to fix:
    • [specific issue]
    • ...

  STRENGTHS — keep these:
    • [what's working well]
    • ...
╚══════════════════════════════════════════╝
```

## Step 4: Apply rewrites (ask user)

Ask: "Should I apply the CRITICAL and IMPORTANT fixes and save a revised version?"

If yes:
- Apply all CRITICAL and IMPORTANT directives to produce a revised chapter
- Preserve the chapter's voice, characters, and plot events — only improve execution
- Save the revised version as `./output/chapter-[N]-revised.md` (never overwrite the original)
- Print a brief diff summary: what paragraphs changed and why

If no: stop here.

## Step 5: Wiki update (if revision introduced new facts)

If the revision added any world facts, character details, or themes not already in the wiki, offer to append them. Follow the same wiki-append format used by `/novel-write`.

## Step 6: Report

```
═══════════════════════════════════════════
  Novel Review Complete
═══════════════════════════════════════════
  Chapter reviewed:  chapter-[N].md
  Overall score:     [score]/100
  Revised file:      chapter-[N]-revised.md  (if created)
═══════════════════════════════════════════
```
