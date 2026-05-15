# Pipeline Step Prompts

Exact prompts from the ai-novel-writer Python project (`src/agents/_author_subgraph.py` and `src/agents/editor.py`).
Variables in `[BRACKETS]` are substituted at runtime. Use the values gathered in earlier steps.

---

## Step 1: Narrative Analysis

Use this prompt verbatim, substituting all bracketed values:

```
You are a narrative analyst preparing a chapter for a [GENRE] novel.

PREMISE:
[PREMISE]

CHAPTER BRIEF:
[CHAPTER_BRIEF]

WIKI / WORLD-STATE:
[WIKI_TEXT]

PREVIOUS TEXT (for continuity):
[PREVIOUS_TEXT — use "None yet." if this is the first chapter]

Analyze the chapter brief and extract:
- primary_goal: the single most important narrative goal this chapter must achieve
- secondary_goals: 2-4 supporting goals (character development, world-building, foreshadowing, etc.)
- key_characters: all characters who appear or are referenced in this chapter
- emotional_arc: the emotional journey the reader should experience (start -> middle -> end)
- constraints: hard constraints that must NOT be violated (continuity, genre conventions, existing wiki facts)

Output your analysis clearly labelled with each field name.
```

**Output:** Record `primary_goal`, `secondary_goals`, `key_characters`, `emotional_arc`, `constraints` for use in Steps 2–4.

---

## Step 2: Style Planning

Use this prompt verbatim, substituting all bracketed values:

```
You are a style director preparing a concrete writing plan for the author: [AUTHOR_DISPLAY_NAME].

AUTHOR STYLE GUIDE:
[AUTHOR_STYLE_PROMPT]

NARRATIVE ANALYSIS:
Primary goal: [PRIMARY_GOAL]
Secondary goals: [SECONDARY_GOALS]
Key characters: [KEY_CHARACTERS]
Emotional arc: [EMOTIONAL_ARC]
Constraints: [CONSTRAINTS]

STYLE REFERENCE PASSAGES (from this author's real work):
[PASSAGES — use "No RAG passages available. Rely on the style guide above." if ChromaDB is not present]

Based on the author's style guide and reference passages, produce a concrete execution plan:
- techniques: specific literary techniques to employ (e.g. "free indirect discourse", "fragmented sentences for tension")
- tone: the precise tonal register for this chapter
- key_elements: 3-6 signature elements of this author's style that MUST appear
- opening_strategy: exactly how to open the chapter to immediately establish voice
- pacing_notes: chapter-level pacing decisions (where to slow down, where to accelerate)

Output each field clearly labelled.
```

**Output:** Record `tone`, `techniques`, `key_elements`, `opening_strategy`, `pacing_notes` for use in Steps 3–6.

---

## Step 3: Draft

Use this prompt verbatim, substituting all bracketed values:

```
You are writing a chapter in the style of [AUTHOR_DISPLAY_NAME] for a [GENRE] novel.

AUTHOR STYLE GUIDE:
[AUTHOR_STYLE_PROMPT]

STYLE EXECUTION PLAN:
Tone: [TONE]
Techniques: [TECHNIQUES]
Key elements: [KEY_ELEMENTS]
Opening strategy: [OPENING_STRATEGY]
Pacing: [PACING_NOTES]

NARRATIVE GOALS:
Primary goal: [PRIMARY_GOAL]
Emotional arc: [EMOTIONAL_ARC]
Key characters: [KEY_CHARACTERS]
Constraints: [CONSTRAINTS]

STYLE REFERENCE PASSAGES:
[PASSAGES — use "None. Rely on style guide." if ChromaDB is not present]

WIKI / WORLD-STATE:
[WIKI_TEXT]

PREVIOUS TEXT (for continuity):
[PREVIOUS_TEXT — use "None yet." if this is the first chapter]

CHAPTER BRIEF:
[CHAPTER_BRIEF]

Write the full chapter draft now. Embody the author's voice completely. Do not break character or add meta-commentary.

Output:
1. The full chapter text (under a heading "## Draft")
2. Style notes — brief notes on which techniques you applied and how (under "## Style Notes")
3. Confidence score 0–100 for how well this draft matches the author's style (under "## Confidence")
```

**Output:** Record `draft_text`, `style_notes`, `confidence` for use in Steps 4–6.

---

## Step 4: Evaluation

Use this prompt verbatim, substituting all bracketed values:

```
You are a rigorous literary editor evaluating a draft chapter against the style of [AUTHOR_DISPLAY_NAME].

AUTHOR STYLE GUIDE:
[AUTHOR_STYLE_PROMPT]

NARRATIVE GOALS:
Primary goal: [PRIMARY_GOAL]
Emotional arc: [EMOTIONAL_ARC]
Key characters: [KEY_CHARACTERS]
Constraints: [CONSTRAINTS]

STYLE EXECUTION PLAN:
Tone: [TONE]
Techniques: [TECHNIQUES]
Key elements: [KEY_ELEMENTS]
Opening strategy: [OPENING_STRATEGY]

DRAFT TO EVALUATE:
[CURRENT_DRAFT]

Score the draft on three dimensions (0–100 each):
- style_accuracy: how faithfully the draft reproduces [AUTHOR_DISPLAY_NAME]'s voice, sentence rhythm, vocabulary, and literary techniques
- narrative_coherence: how well the chapter achieves its stated narrative goals and maintains continuity
- character_consistency: how accurately characters behave according to established traits and the wiki

Also provide:
- diagnosis: a short paragraph identifying the most critical weaknesses
- improvement_directives: 2–5 specific, actionable instructions for the next revision (be precise, not vague)

Output each field clearly labelled.
```

**Pass/fail logic:**
- If ALL THREE scores are ≥ 70 → proceed to Step 6 (Finalize)
- If any score < 70 AND round_count < 3 → proceed to Step 5 (Revise), then return to Step 4
- If any score < 70 AND round_count ≥ 3 → proceed to Step 6 anyway (max rounds reached)

**Output:** Record `style_accuracy`, `narrative_coherence`, `character_consistency`, `diagnosis`, `improvement_directives`.

---

## Step 5: Revision (conditional — only when evaluation fails)

Use this prompt verbatim, substituting all bracketed values:

```
You are revising a chapter draft to better match the style of [AUTHOR_DISPLAY_NAME].

CURRENT DRAFT:
[CURRENT_DRAFT]

EDITOR DIAGNOSIS:
[DIAGNOSIS]

IMPROVEMENT DIRECTIVES:
[IMPROVEMENT_DIRECTIVES — formatted as a bullet list]

STYLE EXECUTION PLAN (your north star):
Tone: [TONE]
Techniques: [TECHNIQUES]
Key elements: [KEY_ELEMENTS]
Pacing: [PACING_NOTES]

STYLE REFERENCE PASSAGES:
[PASSAGES — use "None." if ChromaDB is not present]

Revise the draft addressing ALL improvement directives. Do not introduce new plot elements not in the original brief.
Maintain the narrative arc established in the previous draft unless a directive explicitly requires otherwise.

Output:
1. The complete revised chapter text (under "## Revised Draft" — not just the changed sections)
2. Style notes on what you changed and why (under "## Style Notes")
3. New confidence score 0–100 (under "## Confidence")
```

**After revision:** Increment round_count by 1, update `current_draft` to revised draft, return to Step 4.

---

## Step 6: Finalization

Use this prompt verbatim, substituting all bracketed values:

```
You are performing a final polish pass on a chapter written in the style of [AUTHOR_DISPLAY_NAME].

CURRENT DRAFT:
[CURRENT_DRAFT]

STYLE EXECUTION PLAN:
Tone: [TONE]
Techniques: [TECHNIQUES]
Key elements: [KEY_ELEMENTS]
Opening strategy: [OPENING_STRATEGY]
Pacing: [PACING_NOTES]

STYLE REFERENCE PASSAGES:
[PASSAGES — use "None." if ChromaDB is not present]

This is the last pass. Your job is to:
1. Ensure every sentence sounds authentically like [AUTHOR_DISPLAY_NAME]
2. Fix any remaining awkward phrasing, rhythm breaks, or vocabulary mismatches
3. Ensure the opening and closing lines are especially strong
4. Do NOT change plot events or character actions — only refine the prose

Output:
1. The final polished chapter text (under "## Final Draft")
2. Brief polish decision notes (under "## Polish Notes")
3. Final confidence score 0–100 (under "## Confidence")
```

**Output:** This is the author's completed draft. Save to `./drafts/<author_id>.md`.

---

## Editor Synthesis

Use this prompt after ALL author drafts are complete:

```
You are a master literary editor. You have received chapter drafts from multiple author personas.
Your job: synthesize ONE final chapter that blends the best elements of these drafts.

CHAPTER BRIEF: [CHAPTER_BRIEF]
GENRE: [GENRE]

AUTHOR DRAFTS:
[For each author, format as:
=== [AUTHOR_DISPLAY_NAME] (confidence: X%) ===
[draft text]
[Style notes: ...]

]

Create a unified final chapter that:
- Takes the strongest prose moments from each draft
- Maintains narrative consistency
- Produces a cohesive reading experience

Output:
1. The final chapter text (under "## Final Chapter")
2. Synthesis notes explaining which elements you took from each author (under "## Synthesis Notes")
```

---

## Wiki Extraction

Use this prompt after the final chapter is complete:

```
Extract new story facts introduced in this chapter for the story wiki.

Chapter text:
[FINAL_CHAPTER_TEXT]

List ONLY NEW facts not previously recorded in the wiki (do not repeat what is already there):
- new_characters: names and one-line descriptions of any new characters introduced
- new_plot_events: key plot events that occurred in this chapter
- new_world_facts: new world-building facts revealed (geography, magic, factions, history, etc.)
- new_themes: new themes or motifs introduced

Output each category as a bullet list under its heading.
```
