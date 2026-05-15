# novel-writer

**A multi-author novel-writing pipeline that runs entirely as Claude Code skills — no API keys, no servers, no Python.**

Drop this repo into any folder, open it in [Claude Code](https://claude.com/claude-code), and type `/novel-write`. Fifteen author personas — Hemingway, Tolkien, Le Guin, Sanderson, Pratchett, and eleven others — each draft your chapter through a six-step analyze → plan → draft → evaluate → revise → finalize loop. An editor then synthesizes the strongest elements into a final chapter and updates your story wiki.

## The skills

| Command | What it does |
|---|---|
| `/novel-setup` | Scaffold a new novel: creates `wiki/` with `characters.md`, `plot.md`, `world.md`, `themes.md`. Run once per project. |
| `/novel-write` | Generate a chapter via the 15-author pipeline. Args: `--genre`, `--premise`, `--chapter`, `--authors`, `--previous`. |
| `/novel-review` | Four-pass literary review of a chapter — scored editorial report with specific rewrite directives, optionally applied. |
| `/novel-research` | Research world-building, history, or thematic topics with live web search and ingest findings into the wiki. |
| `/novel-reader` | Render a chapter as a styled HTML page and open it in the browser for proper reading. |
| `/novel-wiki` | Browse, search, and edit the story wiki. |

## Quick start

1. **Clone** this repo into the folder where you want your novel to live:
   ```
   git clone https://github.com/chianglianglin/novel-writer.git my-novel
   cd my-novel
   ```
2. **Open in Claude Code.**
3. **Initialize:** `/novel-setup` — scaffolds the wiki. (The repo ships with a sample "Ashen Crown" wiki you can replace or build from.)
4. **Write a chapter:**
   ```
   /novel-write --genre "epic fantasy" \
                --premise "A blind cartographer discovers maps that predict the future" \
                --chapter "Chapter 1: The cartographer receives a mysterious commission"
   ```
5. **Review and read** the result with `/novel-review` and `/novel-reader`.

## How it works

```
        ┌──────────────────────────────────────────────┐
        │              Chapter brief + wiki            │
        └──────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
       Hemingway           Tolkien        ...13 more
       (6-step loop)    (6-step loop)    (6-step loop)
            │                 │                 │
            └─────────────────┼─────────────────┘
                              ▼
                      ┌──────────────┐
                      │   Editor     │  synthesizes best elements
                      └──────────────┘
                              │
                              ▼
                  output/chapter-N.md  +  wiki update
```

The 15 author personas are: Hemingway, Tolkien, Christie, King, Austen, Márquez, McCarthy, Sanderson, Hobb, Abercrombie, Clarke, Jemisin, Gibson, Le Guin, Pratchett. Each brings a distinct voice — terse and physical, lyrical and mythic, plotted and surprising — and the editor cherry-picks across them so the final chapter is stronger than any single voice could produce alone.

## Repo layout

```
.claude/skills/      the six novel-* skills (the deliverable)
wiki/                sample story bible — the "Ashen Crown" template
drafts/              per-author drafts from previous runs (sample)
output/              finalized chapters from previous runs (sample)
chroma_db/           optional ChromaDB vector store for RAG continuity
```

The `wiki/`, `drafts/`, `output/`, and `chroma_db/` directories contain a worked example. Feel free to delete them when starting your own novel — `/novel-setup` will rebuild `wiki/` from scratch.

## Requirements

- [Claude Code](https://claude.com/claude-code) (any model).
- That's it. No API keys. No Python. No Node. No external runtime — the skills are the whole project.

## License

MIT — see [LICENSE](./LICENSE).

## Credits

Inspired by the original Python `ai-novel-writer` project; this version ports the pipeline natively into Claude Code skills so it runs anywhere Claude Code does, without a separate runtime.
