---
name: novel-write
description: Generate a novel chapter using a multi-author pipeline. Each of 15 author personas drafts the chapter through a 6-step analyze→plan→draft→evaluate→revise→finalize loop, then an editor synthesizes the best elements into a final chapter and updates the story wiki.
---

# Novel Write

Run the full multi-author chapter generation pipeline. Replicates the ai-novel-writer Python project natively in Claude, with optional ChromaDB RAG if the Python project is available.

## Skill directory

This skill lives alongside two companion files in the same directory:
- `authors.md` — all 15 author style guides (read this at Step 2)
- `prompts.md` — the exact 6-step pipeline prompts (read this at Step 2)

The skill base directory is shown in the `Base directory for this skill:` line at the top of this file when invoked. Use it to find the companion files.

---

## Step 1: Gather inputs

Parse arguments passed to `/novel-write`. Accept in any order:

| Arg | Description | Default |
|-----|-------------|---------|
| `--genre` or first positional | Story genre | ask user |
| `--premise` | One-paragraph story premise | ask user |
| `--chapter` | Chapter brief (what this chapter covers) | ask user |
| `--authors` | Comma-separated author IDs to use | all 15 |
| `--wiki` | Path to wiki directory | `./wiki/` |
| `--previous` | Path to previous chapter file for continuity | none |

If `--genre`, `--premise`, or `--chapter` are missing, ask the user for them before proceeding.

**Author IDs (valid values for `--authors`):**
hemingway, tolkien, christie, king, austen, marquez, mccarthy, sanderson, hobb, abercrombie, clarke, jemisin, gibson, leguin, pratchett

**Example invocations:**
```
/novel-write --genre "epic fantasy" --premise "A blind cartographer discovers maps that predict the future" --chapter "Chapter 1: The cartographer receives a mysterious commission"
/novel-write --genre noir --premise "..." --chapter "..." --authors hemingway,mccarthy,king
```

---

## Step 2: Load companion files and wiki

Read the following files using the Read tool:

1. **`<skill_base_dir>/authors.md`** — store all author style guides in memory
2. **`<skill_base_dir>/prompts.md`** — store all pipeline prompts in memory
3. **`<wiki_dir>/characters.md`** — story character roster
4. **`<wiki_dir>/plot.md`** — plot events so far
5. **`<wiki_dir>/world.md`** — world-building facts
6. **`<wiki_dir>/themes.md`** — themes and motifs

Concatenate wiki files into a single `WIKI_TEXT` block:

```
=== CHARACTERS ===
<characters.md content>

=== PLOT EVENTS ===
<plot.md content>

=== WORLD FACTS ===
<world.md content>

=== THEMES ===
<themes.md content>
```

If a wiki file does not exist, use `"[Not yet established]"` for that section.

If `--previous` was provided, read that file as `PREVIOUS_TEXT`. Otherwise set `PREVIOUS_TEXT = "None yet."`.

---

## Step 3: Check for ChromaDB (hybrid RAG)

Run this PowerShell check via Bash:
```powershell
Test-Path "F:\AI Stock Projects\ai-novel-writer-no-api\chroma_db"
```

- If output is `True` → **RAG available**. For each author in Step 4, query ChromaDB for style passages (see sub-step below).
- If output is `False` → **RAG unavailable**. Set `PASSAGES = "No RAG passages available. Rely on the style guide above."` for all authors.

**RAG query (run once per author when ChromaDB is available):**

Collection names are the plain author IDs (`hemingway`, `sanderson`, etc.) — each has 20 style-example passages with metadata (scene_type, emotional_tone, techniques, book).

Use Bash to run (save as a temp script or inline via PowerShell):
```python
# Save to a temp file and run: python rag_query.py <author_id> "<chapter_brief>"
import sys, warnings
warnings.filterwarnings("ignore")
author_id = sys.argv[1]
chapter_brief = sys.argv[2]

try:
    import chromadb
    client = chromadb.PersistentClient(path=r"F:\AI Stock Projects\ai-novel-writer-no-api\chroma_db")
    col = client.get_collection(author_id)
    try:
        # Try semantic search (requires OPENAI_API_KEY in environment)
        results = col.query(query_texts=[chapter_brief], n_results=3)
        docs = results.get("documents", [[]])[0]
    except Exception:
        # Fallback: return 3 random docs without semantic ranking
        result = col.get(limit=3, include=["documents"])
        docs = result.get("documents", [])
    print("\n\n---\n\n".join(docs) if docs else "No passages found.")
except Exception as e:
    print(f"RAG unavailable: {e}")
```

Store the output as `PASSAGES` for that author. If an exception occurs or output starts with "RAG unavailable", fall back to `"No RAG passages available. Rely on the style guide above."`

---

## Step 4: Run the 6-step pipeline for each author

Process authors **one at a time** in order (order 1–15, or the subset specified by `--authors`).

For **each author**, do the following sub-steps using the prompts from `prompts.md`. Substitute all `[BRACKET]` variables with the actual values gathered so far.

### 4a. Announce

Print: `\n--- [Author Display Name] (order N) ---`

If RAG is available, query ChromaDB for this author's `PASSAGES` now (see Step 3 RAG query).

### 4b. Step 1 — Narrative Analysis

Use the **Step 1: Narrative Analysis** prompt from `prompts.md`.
Substitute: `[GENRE]`, `[PREMISE]`, `[CHAPTER_BRIEF]`, `[WIKI_TEXT]`, `[PREVIOUS_TEXT]`.

Record: `primary_goal`, `secondary_goals`, `key_characters`, `emotional_arc`, `constraints`.

**Note:** The narrative analysis is the same for all authors — you may run it once and reuse across authors, or run it per-author. Running it once is more efficient.

### 4c. Step 2 — Style Planning

Use the **Step 2: Style Planning** prompt from `prompts.md`.
Substitute: `[AUTHOR_DISPLAY_NAME]`, `[AUTHOR_STYLE_PROMPT]` (from `authors.md`), the narrative analysis fields, `[PASSAGES]`.

Record: `tone`, `techniques`, `key_elements`, `opening_strategy`, `pacing_notes`.

### 4d. Step 3 — Draft

Use the **Step 3: Draft** prompt from `prompts.md`.
Substitute: all style plan fields, all narrative analysis fields, `[AUTHOR_STYLE_PROMPT]`, `[GENRE]`, `[CHAPTER_BRIEF]`, `[WIKI_TEXT]`, `[PREVIOUS_TEXT]`, `[PASSAGES]`.

Record: `current_draft`, `style_notes`, `confidence`.
Set `round_count = 0`.

### 4e. Step 4 — Evaluate

Use the **Step 4: Evaluation** prompt from `prompts.md`.
Substitute: `[AUTHOR_DISPLAY_NAME]`, `[AUTHOR_STYLE_PROMPT]`, narrative analysis fields, style plan fields, `[CURRENT_DRAFT]`.

Record: `style_accuracy`, `narrative_coherence`, `character_consistency`, `diagnosis`, `improvement_directives`.

**Check pass/fail:**
- ALL THREE scores ≥ 70 → skip to 4g (Finalize)
- Any score < 70 AND `round_count` < 3 → go to 4f (Revise)
- Any score < 70 AND `round_count` ≥ 3 → go to 4g anyway (max rounds reached)

Print scores: `  Scores: style=[X] coherence=[Y] character=[Z]`

### 4f. Step 5 — Revise (conditional)

Use the **Step 5: Revision** prompt from `prompts.md`.
Substitute: `[AUTHOR_DISPLAY_NAME]`, `[CURRENT_DRAFT]`, `[DIAGNOSIS]`, `[IMPROVEMENT_DIRECTIVES]` (as bullet list), style plan fields, `[PASSAGES]`.

Update `current_draft` to the revised draft.
Increment `round_count` by 1.
Print: `  Revising (round [round_count])...`

Return to Step 4e (Evaluate).

### 4g. Step 6 — Finalize

Use the **Step 6: Finalization** prompt from `prompts.md`.
Substitute: `[AUTHOR_DISPLAY_NAME]`, `[CURRENT_DRAFT]`, style plan fields, `[PASSAGES]`.

Record `final_draft`, `final_style_notes`, `final_confidence`.

### 4h. Save author draft

Create `./drafts/` directory if it does not exist.
Write the final draft to `./drafts/<author_id>.md`:

```markdown
# [Author Display Name] Draft

**Confidence:** [final_confidence]%

**Style Notes:** [final_style_notes]

---

[final_draft]
```

Print: `  ✓ [Author Display Name] — confidence: [final_confidence]%`

---

## Step 5: Editor synthesis

Once ALL selected authors have completed Step 4, run the **Editor Synthesis** prompt from `prompts.md`.

Substitute:
- `[CHAPTER_BRIEF]` — the chapter brief
- `[GENRE]` — the genre
- Author drafts block: format each as `=== [Author Display Name] (confidence: X%) ===\n[final_draft]\n[Style notes: final_style_notes]`

Record `final_chapter_text` and `synthesis_notes`.

**Save the final chapter:**

Determine the next chapter number by counting files in `./output/`:
```powershell
(Get-ChildItem -Path "./output/" -Filter "chapter-*.md" -ErrorAction SilentlyContinue | Measure-Object).Count + 1
```

Create `./output/` directory if it does not exist.
Write to `./output/chapter-<N>.md`:

```markdown
# Chapter [N]

*Genre: [genre] | Chapter Brief: [chapter_brief]*

---

[final_chapter_text]

---

## Synthesis Notes

[synthesis_notes]

## Author Drafts

- [Author1 Display Name]: `./drafts/<author1_id>.md` (confidence: X%)
- [Author2 Display Name]: `./drafts/<author2_id>.md` (confidence: X%)
[... all selected authors ...]
```

---

## Step 6: Update wiki

Run the **Wiki Extraction** prompt from `prompts.md`.
Substitute `[FINAL_CHAPTER_TEXT]` with `final_chapter_text`.
Substitute context about what is already in the wiki so Claude doesn't duplicate.

For each non-empty list returned:
- **new_characters** → append each entry to `<wiki_dir>/characters.md`
- **new_plot_events** → append each entry to `<wiki_dir>/plot.md`
- **new_world_facts** → append each entry to `<wiki_dir>/world.md`
- **new_themes** → append each entry to `<wiki_dir>/themes.md`

Append with a section header like:
```markdown

## After Chapter [N]

- [entry 1]
- [entry 2]
```

---

## Step 7: Report completion

Print:

```
═══════════════════════════════════════════
  Novel Write Complete
═══════════════════════════════════════════
  Final chapter: ./output/chapter-[N].md
  Author drafts: ./drafts/
  Wiki updated:  [wiki_dir]/

  Authors processed: [list with confidence scores]
═══════════════════════════════════════════
```

---

## Error handling

- If a wiki file is missing, continue — use `"[Not yet established]"` for that section.
- If ChromaDB query fails, continue without passages — do not abort.
- If any author step fails, record the error, skip that author, and continue with the next.
- If the `./drafts/` or `./output/` directory cannot be created, report the error and stop.

---

## Notes on efficiency

- Narrative analysis (Step 1) produces the same result for all authors. You may run it once before the author loop and reuse the result for all authors.
- Each author's Steps 2–6 are independent of other authors and depend only on the shared narrative analysis.
- RAG queries are per-author and should be run immediately before that author's pipeline begins.
