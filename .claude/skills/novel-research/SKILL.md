---
name: novel-research
description: Research any world-building, historical, or thematic topic using live web search and ingest findings into the story wiki.
---

# Novel Research

Research any world-building, historical, or thematic topic using live web search and ingest findings into the story wiki.

## Step 1: Parse input

Accept a free-form topic as the argument. If none given, ask the user what to research.
Store as `TOPIC`.

## Step 2: Load wiki context

Read `./wiki/world.md`, `./wiki/themes.md`, `./wiki/characters.md`, `./wiki/project.md`.
Store as `EXISTING_WIKI` to avoid duplicating known facts.

## Step 3: Web research

Use WebSearch with 2-3 different query phrasings of TOPIC to find concrete factual detail, historical analogs, and how writers have handled the topic.

Use WebFetch to read the 3 most relevant URLs in full.

Synthesize findings into three categories:

**World Facts** — sensory and physical details that could ground scenes (look, sound, smell, feel, human behavioral response)

**Thematic Resonances** — how this topic connects to Ashen Crown's existing themes (reference `./wiki/themes.md`)

**Craft Notes** — techniques and approaches from other writers worth knowing

## Step 4: Present and confirm

Show the synthesized findings to the user with sources listed.
Ask: "Should I add any of these to the wiki? Say 'all', 'just world facts', 'just themes', or specify."

## Step 5: Ingest into wiki

Append approved content to the correct files under a `## Research: [TOPIC]` header.
- World Facts → `./wiki/world.md`
- Thematic Resonances + Craft Notes → `./wiki/themes.md`
- Character-relevant facts → also `./wiki/characters.md`

## Step 6: Report

Print completion summary showing topic, sources read, and wiki files updated.
New wiki facts will automatically enrich the next `/novel-write` run.
