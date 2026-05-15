---
name: novel-reader
description: Convert a generated chapter from markdown into a beautifully styled HTML page and open it in the browser for proper reading.
---

# Novel Reader

Convert a generated chapter from markdown into a beautifully styled HTML page and open it in the gstack browser for proper reading.

## Step 1: Find chapter

Accept `--chapter N` or a file path. If no argument, use the most recent file in `./output/` by modification time.

Read the chapter markdown. Read `./wiki/project.md` for the novel title and genre.

## Step 2: Generate HTML

Invoke the `design-html` skill with this brief:

"Convert this chapter of [TITLE] (dark epic fantasy) into a standalone HTML reading page. Design: aged parchment background (#f5efe0), dark ink text (#1a1209), serif body font (Georgia or similar), generous line-height (1.9), max-width 680px centered. Chapter number as a large styled header. Drop-cap on the first letter of the first paragraph. Pull the most powerful single line from the chapter and display it as a styled pull quote mid-page. No navigation, no UI chrome — just the chapter as a beautiful reading experience. Fully self-contained HTML with all CSS inline."

Pass the full chapter text as the content to style.

## Step 3: Save HTML

Create `./output/html/` if it does not exist.
Save the generated HTML to `./output/html/chapter-[N].html`.

## Step 4: Open in browser

Use the `open-gstack-browser` skill to launch the browser, then navigate to the saved HTML file using its absolute path as a `file://` URL.

If the browser is already open, navigate directly.

Take a screenshot to confirm the page rendered correctly. Show it to the user.

## Step 5: Report

Tell the user:
- Which chapter was rendered
- Where the HTML file was saved
- Confirm the browser is open and showing the chapter

If the rendering looks wrong (broken layout, missing styles), invoke `design-html` again with corrections.
