---
name: cloudhour-deck
description: |
  Fills the CloudHour round-table PowerPoint template (Microsoft EMS Community
  monthly session) on a dated copy: replaces every [bracketed] placeholder on
  the slides and in the speaker notes, auto-fills what it can, researches topics,
  asks in chat for the rest, and loops until no placeholders remain on visible
  slides. Use when the user asks to "fill the CloudHour deck", "prepare
  CloudHour", "new CloudHour session", "fill the CloudHour template" or
  "CloudHour November". Do NOT use for any other deck or for building
  presentations from scratch — use the pptx skill instead.
metadata:
  category: productivity
  icon: SlideTextEdit
---

# CloudHour Deck

## Overview
Turns `CloudHour - Template.pptx` into a ready-to-present session deck named
`CloudHour - [Mon YYYY].pptx`. Every placeholder is text in square brackets
(regex `\[[^\]]+\]`) on a slide or in its speaker notes. The skill fills what it
can derive, drafts topic content from research, asks the user for the rest in
grouped batches, and repeats until the stop condition is met. All slide content,
notes and questions are in **English**.

## When to Use
- "Fill the CloudHour deck for 18 November 2026, co-host Jane Doe, topics: MC123456 and Windows Hotpatch"
- "Prepare CloudHour December" / "New CloudHour session" / "Fill the CloudHour template"
- Continuing an in-progress CloudHour deck that still has `[...]` placeholders

## When NOT to Use
- Any deck other than the CloudHour round-table template — use the pptx skill
- Creating a new presentation from scratch or redesigning the template — use the pptx skill
- Writing a CloudHour invitation e-mail or Teams post — use the mail or teams skill

## Quick Start
```
User: "Fill the CloudHour deck for 18 November 2026, co-host Jane Doe,
       topics: MC123456 and Windows Hotpatch"
1. Locate the template → copy to "CloudHour - Nov 2026.pptx" → open the copy live
2. Scan all slides + notes → placeholder inventory per slide
3. Auto-fill date, [Month Year], [Number of topics], host name, numbering
4. Research co-host + topics → show drafts per slide batch → write after approval
5. Ask for remaining placeholders, grouped per slide; offer fill / hide / delete / skip
6. Re-scan → report remaining placeholders → loop until done
7. Final QA (render, overflow, footers, numbering) → deliver summary
```

## Core Instructions

Load the **pptx** skill before the first edit; it owns the live-edit op vocabulary.

### Step 1 — Locate and copy (never edit the template)
1. Prefer a template the user attached or named (`input/` first). Otherwise find
   `CloudHour - Template.pptx` with SearchDrive / SearchM365 in the user's OneDrive
   (known location: `Documenten/Cowork/Tasks/placeholders-vervangen-in-powerpoint-2026-10-31/output/`).
   If several copies exist, ask which one to use.
2. Determine the session date. CloudHour is on a **Wednesday**. If the user gave no
   date, propose the next likely Wednesday and ask for confirmation. If the given
   date is not a Wednesday, point that out and ask.
3. Bring the template into the workspace (ReadFileContent / download to `working/`)
   and create the copy with CopyArtifact (`surface="output"`) named
   `CloudHour - [Mon YYYY].pptx` (e.g. `CloudHour - Nov 2026.pptx`). Use the returned
   `file_N` handle for every later read and edit on `surface="live"`.
   If a file with that name already exists, ask: continue that deck, or create
   `CloudHour - Nov 2026 (2).pptx`. Never point an edit at the template's handle.

### Step 2 — Scan dynamically (every round)
1. GetArtifactModel(surface="live", path=file_N) → all slides, shapes and text.
   Read every slide's speaker notes as well (from the model, or from a rendered copy
   via `renderPath` when the model omits notes).
2. Build an inventory per slide: slide number, **eyebrow** (small caps line) and
   **large title** (use both to understand the slide's purpose), hidden yes/no,
   placeholders on the slide (shape index + bracketed text) and in the notes.
3. Only bracketed text is a placeholder. Fixed titles, eyebrows and body text are
   never touched (e.g. "What is CloudHour", "Housekeeping", radar column headers).
4. Never reuse shape or slide indices from an earlier round: re-scan after every
   structural change (duplicate, delete, reorder, image swap).

Typical template roles (for recognition only — always derive from the scan):
cover (`WED · [DATE]`), today's co-host, fixed intro slides, links to share,
agenda "On the table" (topic cards + `[Number of topics]`), radar "Next 90 days"
(12× `[DATE]` + `[Radar item N]`), per-topic blocks (topic detail + "Let us talk"
round-table, optional bullet summary / image slide / quiz), open floor, thank you.
Footer on most slides: `CloudHour · Microsoft EMS Community · [Month Year]`.

### Step 3 — Auto-fill without asking
- `[Month Year]` → `Nov 2026` style from the session date (every footer).
- Cover `[DATE]` → `18 NOV 2026` (or the user's preferred format).
- `[Number of topics]` → number of topics the user gave.
- `TOPIC N` eyebrows, agenda numbers and `[Topic N …]` labels → renumber
  consistently with the final topic order.
- Notes: date, `[HOST NAME]` (the user, via GetMyDetails, unless told otherwise),
  `[CO-HOST NAME]` from the user's message.
Write these in one batch: EditArtifact(surface="live") with `set_text` and
`keepFormat: true`, **one set_text per shape per batch**; notes via `set_notes`
(preserve the rest of the note text, replace only the placeholders).

### Step 4 — Research and draft (show before writing)
- **Co-host**: web_search (LinkedIn, MVP profile at mvp.microsoft.com, personal blog)
  to PROPOSE job title, "known from", badges and websites. Show the proposal; write
  only after the user confirms.
- **Topics** (Message Center ID, Learn URL or feature name): research with
  microsoft_docs_search / microsoft_docs_fetch, web_search and the Message Center
  roadmap pages. Draft per topic: workload · area, title, subtitle, Heading/Text 1–4,
  deadline date + summary, who is affected, source, 4 discussion questions, short
  name, radar items, and notes with summary + source URLs.
- Present drafts **per slide batch** (one topic block at a time) as a compact table
  `placeholder → proposed text`; write after the user approves or edits.
- Every date, deadline, MC ID and URL must come from a tool result. If research
  finds nothing, leave the placeholder and ask.

### Step 5 — Ask for the rest in chat
Group open placeholders per slide, a few slides per message, in this format:

```
Slide 5 — "Links to share"
  Open: [Link 1 title], [Link 1 URL or description], … [Link 7 …]
  Needed: up to 7 links to share in the chat (title + URL)
  Options: fill · hide this slide · delete this slide · skip for now
```
Accept free-form answers (pasted text, links, lists) and map them to placeholders;
confirm the mapping when it is ambiguous. Unused optional items (e.g. Link 6–7)
→ ask whether to delete the empty shapes.

### Step 6 — Structure changes
- More topics or a longer topic: `duplicate_slide` the topic detail + round-table
  pair (or the optional bullet summary / image / quiz slide) and place it in order.
- Fewer topics: delete or hide unused blocks and remove the unused agenda cards
  (ask first when unsure).
- Renumber all `TOPIC N` labels and agenda cards, update `[Number of topics]`, then
  **re-scan**. If hiding a slide is not possible with the available edit ops, ask
  the user to hide it in PowerPoint, or offer deletion instead.

### Step 7 — Images
For `[Badge logo]` and `[Image or screenshot]` boxes: ask the user to attach the
image, or propose a public image found via search (e.g. the official MVP logo) and
get confirmation. Insert it at the placeholder box's position and size (`add_image`
/ `replace_image`), then `delete_shape` the placeholder box. No image → ask: delete
the box or hide the slide.

### Step 8 — Loop and stop
After each round, re-scan and report:

| Slide | Heading | Remaining placeholders |
|---|---|---|

Stop when **no `[...]` remains on any visible slide or its notes**, or the user
explicitly chose to hide/delete the slides that still have them. Hidden slides
with placeholders do not block completion. Then remove the `[TEMPLATE STRUCTURE]`
block from the agenda notes.

### Step 9 — Final QA and delivery
1. GetArtifactModel(render=true) → RenderArtifact per slide → check for overflowing
   or badly wrapping text; shorten text that overflows (keep titles short).
2. Verify every footer reads `CloudHour · Microsoft EMS Community · Nov 2026`,
   topic numbering is consecutive, agenda count matches the topic blocks.
3. Final scan must return zero placeholders on visible slides.

## Output
Final chat message (short):
- The deck, referenced by its file handle (`CloudHour - Nov 2026.pptx`)
- **Filled automatically**: date, footers, numbering, notes, …
- **Researched & approved**: co-host profile, topic content with sources
- **From the user**: links, images, answers, …
- **Hidden / deleted slides** and any hidden slides that still contain placeholders

## Guardrails
- **Never modify the original template** — every edit targets the dated copy's handle.
- Never fabricate dates, deadlines, Message Center IDs, prices, names or URLs —
  research them or ask; only URLs that appeared in tool results go into notes.
- Show co-host and topic drafts before writing; destructive changes (delete slide,
  delete shape) only after the user chose them.
- Keep the template's formatting: `set_text` with `keepFormat: true`, one set_text
  per shape per batch; never restyle.
- English only for slide content, notes and questions.
- If a live edit batch fails midway, re-read the model before retrying.
