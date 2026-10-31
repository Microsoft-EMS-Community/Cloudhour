---
name: cloudhour-deck
description: |
  Fills the CloudHour round-table PowerPoint template (Microsoft EMS Community
  monthly session) on a dated copy: replaces every [bracketed] placeholder on
  the slides and in the speaker notes, auto-fills what it can, researches
  topics, then walks the deck slide by slide in order, asking in chat until no
  placeholders remain on visible slides — optionally from a CloudHour monthly
  markdown the user provides (link, uploaded .md or pasted text). Use when the
  user asks to "fill the CloudHour deck", "prepare CloudHour", "new CloudHour
  session", "fill the CloudHour template" or "CloudHour November". Do NOT use
  for any other deck or for building presentations from scratch — use the
  pptx skill instead. Uses only what the user provides plus public-web research
  on it; never searches the user's mailbox, calendar, Teams or other files.
metadata:
  category: productivity
  icon: SlideTextEdit
---

# CloudHour Deck

## Overview
Turns `CloudHour - Template.pptx` into a ready-to-present session deck named
`CloudHour - [Mon YYYY].pptx`. Every placeholder is text in square brackets
(regex `\[[^\]]+\]`) on a slide or in its speaker notes. The skill fills what it
can derive, drafts topic content from research, then walks the deck **strictly
in slide order, one slide at a time**, until the stop condition is met. All slide
content, notes and questions are in **English**.

**Optional input — a CloudHour monthly markdown.** The skill can also use a
monthly CloudHour markdown file, but **only when the user provides one** — in
the request or at any later moment in the chat — as a URL (GitHub blob or raw
link), an attached/uploaded `.md` file, or pasted markdown text. It is never a
default or automatic source: never derive or guess a URL from the session
month, never probe a repository for it, never ask "do you have the link?", and
do not mention any repository unless the user brings it up. Without a markdown
the skill works exactly as described below (auto-fill + research + ask per
slide, in chronological order).

## Allowed sources (strict — applies to every step)
This skill may get information from **only** these three sources:

**ALLOWED**
1. **The user** — what they give in the request or in the chat: text, answers,
   pasted markdown, links, and files they upload/attach (e.g. a `.md` or images).
2. **The public web, only to research what the user provided** — fetch the URLs
   the user gave (incl. a provided CloudHour markdown link and the topic links
   inside it), visit those websites and follow their documentation, and run web
   searches / Microsoft Learn docs lookups about the topics, Message Center IDs,
   features or co-host the user named. Opening a user-provided URL in the local
   browser as a fallback is allowed.
3. **The deck files themselves** — finding the CloudHour template file by name in
   OneDrive/workspace, copying it, and reading/editing the working copy.

**NOT ALLOWED — never, not even as a fallback**
- Searching or reading the user's mailbox/e-mail, calendar, meetings or
  transcripts, Teams chats/channels, OneNote, or SharePoint/OneDrive documents
  other than the CloudHour template/working copy and files the user explicitly
  attaches.
- People/org directory lookups (including the user's own profile), prior Copilot
  conversations, memories, or any enterprise/Microsoft 365 search (Copilot / M365
  search, SQL over M365 data) to find session info, co-host details, dates or topics.
- Opening other documents found while looking for the template — at most list
  them by name as candidates (e.g. to warn that one is an old filled deck).

**Missing information → ASK the user in chat**, at the slide where it is needed.
Never fill a gap from the user's Microsoft 365 data. If a step below seems to need
such a source, ask instead.

## When to Use
- "Fill the CloudHour deck for 18 November 2026, co-host Jane Doe"
- "Prepare CloudHour December" / "New CloudHour session" / "Fill the CloudHour template"
- "Fill the CloudHour deck using <link to a CloudHour monthly .md>" or with an
  attached/pasted markdown
- Continuing an in-progress CloudHour deck that still has `[...]` placeholders

## When NOT to Use
- Any deck other than the CloudHour round-table template — use the pptx skill
- Creating a new presentation from scratch or redesigning the template — use the pptx skill
- Writing a CloudHour invitation e-mail or Teams post — use the mail or teams skill

## Quick Start
```
User: "Fill the CloudHour deck for November 2026"          (no markdown)
1. Locate the template by file name → copy to "CloudHour - Nov 2026.pptx" → open the copy live
2. Scan all slides + notes → placeholder inventory per slide
3. Auto-fill the session date (first Wednesday of the month, e.g. Wed 4 Nov 2026),
   [Month Year], numbering, and names the user already gave
4. Public-web research on user-provided topics/links/names → drafts (proposals only)
5. Opening message, then start the walk at slide 1 (ask co-host and host name in
   chat if not given — never look them up in mail/calendar/Teams)
6. Walk slide 1 → last, one slide at a time: show → confirm/change/hide/delete/skip
   → write → re-scan → next slide; second pass over "skip for now" slides
7. Final QA (render, overflow, footers, numbering) → deliver summary

User provides a markdown (in the request, or mid-walk):
→ read it (Step 1) → map it onto the deck → show what it fills
→ continue the walk from slide 1 (at start) or from the current slide (mid-walk)
```

## Core Instructions

Load the **pptx** skill before the first edit; it owns the live-edit op vocabulary.

### Step 1 — Optional: read a user-provided monthly markdown
Run this step **only if the user has provided a markdown** (now or later in the
chat). If none was provided, skip it entirely and do not ask for one.
1. **Recognise the input.** A URL to a `.md` file (GitHub `blob` or
   `raw.githubusercontent.com` link), an attached/uploaded `.md` file (check
   `input/` first), or markdown text pasted in the chat. Never search the user's
   mail, Teams, OneDrive or M365 for a markdown or its content.
2. **Fetch a link, in this order:** convert a `github.com/.../blob/...` link to its
   raw form (`https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>`);
   (a) web_fetch on the raw URL; (b) if that returns nothing or an error, open the
   raw URL in the local browser and read the page text; (c) if both fail, say so and
   ask the user to paste the file content — meanwhile continue the normal walk.
   Never fall back to a Microsoft 365 / Copilot search.
   An uploaded file is read directly; pasted text is used as-is.
3. **Parse tolerantly** — match sections by heading text (case-insensitive) and
   bullets by their bold label (`**Title:**`, `**Date:**`, …), not by line number.
   Layout and slide mapping: [references/source-file-mapping.md](references/source-file-mapping.md).
   Header/month, "Today's Agenda" topics (Title / Workload Area / Date / Link) →
   agenda cards + topic blocks; "Links to Share" (Link N: Title / URL / Description)
   → links slide + notes; Open Floor → open-floor slide.
4. **Respect what the file itself says.** If it instructs e.g. "Omit unfilled
   entries" or contains `TBD` / empty values, never write "TBD" or the empty entry
   into the deck; ask once (when the walk reaches it) whether to drop or hide that
   card/block.
5. **Status.** If the file has a `**Status:**` line, just mention its value (e.g.
   "Status: Draft") — it is not a gate.
6. **Unknown sections** → map by heading meaning to a matching slide if one exists;
   sections that match no slide are listed to the user once, then ignored.
7. **Provided mid-walk** → parse it, map it onto the remaining slides **and** onto
   earlier slides that still hold placeholders, show in one message what it would
   fill (and any structural change it implies, e.g. fewer topics), apply the
   auto-fill values after a short confirm, re-scan, and **continue the walk from
   the current slide** — never restart at slide 1 and never jump ahead. Earlier
   slides that gain proposals are revisited in the second pass.

### Step 2 — Locate and copy (never edit the template)
1. Prefer a template the user attached or named (`input/` first). Otherwise find
   the file **by name only** — a OneDrive file search (SearchDrive) for
   `CloudHour - Template.pptx` (known location:
   `Documenten/Cowork/Tasks/placeholders-vervangen-in-powerpoint-2026-10-31/output/`).
   Do not use an enterprise/M365 content search, and do not open any other file
   the search returns. If several CloudHour files match, list them by name and ask
   which one to use — flag a candidate whose name suggests a filled month deck
   (e.g. `CloudHour - Oct 2026.pptx`) as an old deck, not the template. If the
   template cannot be found, ask the user to attach it or give its location.
2. Determine the session date by rule. CloudHour is **always on the first
   Wednesday of the session month** (e.g. November 2026 → Wednesday 4 November
   2026; compute it, never guess). Use it without asking. Only if the user states
   a different date (in the request or later in chat) use theirs — and if that
   date is not a Wednesday, point that out and ask. If no month was given, take
   the month from the user's request or a provided markdown; otherwise ask for the
   month in chat. Never check the calendar, meeting invites or e-mail for the date.
3. Bring the template into the workspace (ReadFileContent / download to `working/`)
   and create the copy with CopyArtifact (`surface="output"`) named
   `CloudHour - [Mon YYYY].pptx` (e.g. `CloudHour - Nov 2026.pptx`). Use the returned
   `file_N` handle for every later read and edit on `surface="live"`.
   If a file with that name already exists, ask: continue that deck, or create
   `CloudHour - Nov 2026 (2).pptx`. Never point an edit at the template's handle.

### Step 3 — Scan dynamically (after every write)
1. GetArtifactModel(surface="live", path=file_N) → all slides, shapes and text.
   Read every slide's speaker notes as well (from the model, or from a rendered copy
   via `renderPath` when the model omits notes).
2. Build an inventory per slide: slide number, **eyebrow** (small caps line) and
   **large title** (use both to understand the slide's purpose), hidden yes/no,
   placeholders on the slide (shape index + bracketed text) and in the notes.
3. Only bracketed text is a placeholder. Fixed titles, eyebrows and body text are
   never touched (e.g. "What is CloudHour", "Housekeeping", radar column headers).
4. Never reuse shape or slide indices from an earlier scan: re-scan after every
   write and every structural change (duplicate, delete, reorder, image swap).

Typical template roles (for recognition only — always derive from the scan):
cover (`WED · [DATE]`), today's co-host, fixed intro slides, links to share,
agenda "On the table" (topic cards + `[Number of topics]`), radar "Next 90 days"
(12× `[DATE]` + `[Radar item N]`), per-topic blocks (topic detail + "Let us talk"
round-table, optional bullet summary / image slide / quiz), open floor, thank you.
Footer on most slides: `CloudHour · Microsoft EMS Community · [Month Year]`.

### Step 4 — Auto-fill without asking
- **Session date** → first Wednesday of the month (Step 2.2): cover `WED · [DATE]`
  (e.g. `WED · 4 NOV 2026`, matching the template's casing), the date in the
  speaker notes (e.g. "Wednesday 4 November 2026").
- `[Month Year]` → `Nov 2026` style from the session month (every footer).
- `TOPIC N` eyebrows, agenda numbers and `[Topic N …]` labels → renumber
  consistently with the final topic order.
- Notes: date, `[HOST NAME]` and `[CO-HOST NAME]` — only when the user stated
  them in the request or chat. Otherwise ask at the first slide that needs them;
  never look the host up in the profile/people directory or the co-host in mail,
  calendar or Teams.
- **Only when a markdown was provided:** `[Number of topics]` / agenda intro → the
  number of filled topics (e.g. "Five changes hitting your tenant. …"), agenda card
  title / workload · area / date pill, topic eyebrow + title + deadline date,
  links-slide titles and short URLs, open-floor text — see the mapping reference.
  Without a markdown these come from the user and research during the walk.
Write these in one batch: EditArtifact(surface="live") with `set_text` and
`keepFormat: true`, **one set_text per shape per batch**; notes via `set_notes`
(preserve the rest of the note text, replace only the placeholders). These values
are still shown to the user, per slide, during the walk.

### Step 5 — Research and draft (before the walk)
Research uses the **public web only** (web fetch, web search, Microsoft Learn docs
search/fetch, local browser for a user-provided URL) and only about what the user
provided. Never use Copilot / M365 search, mail, Teams, OneNote, SharePoint or
people lookups here.
- **Topics**: research each topic, MC ID or feature the user named (or, when a
  markdown was provided, each topic's `**Link:**` as primary source; Learn pages via
  microsoft_docs_fetch). Supplement with microsoft_docs_search, web_search and
  public Message Center / roadmap pages. Draft per topic: subtitle, Heading/Text 1–4,
  deadline summary, who is affected, `Source: <site name>` (e.g. Microsoft Learn /
  Tech Community / 365Fanatic), 4 round-table discussion questions, short name, and
  notes with a summary and the source URLs. Topics not yet known → ask at the
  agenda slide.
- **Radar "Next 90 days"**: draft only items whose date appears in a fetched
  source; otherwise ask at that slide.
- **Co-host**: only the name or profile link the user gave — public web_search /
  fetch of that link (LinkedIn, MVP profile at mvp.microsoft.com, personal blog) to
  PROPOSE job title, "known from", badges and websites. If the name is common or
  several profiles match, ask for a LinkedIn/MVP link first. No co-host given →
  ask at the co-host slide; never search mail, Teams, calendar or the directory.
- Drafts are proposals: nothing researched is written until the user confirms it
  on its slide during the walk.
- Every date, deadline, MC ID and URL must come from the user, a provided markdown
  or a public-web result on what the user provided. If nothing is found, leave the placeholder and ask at that slide.

### Step 6 — Strict chronological walk-through
After the scan, auto-fill and research, send **one opening message**: what was
auto-filled; if a markdown was provided — its status (if stated), what was mapped
(e.g. "5 topics, 7 links, open floor"), TBD/empty entries, unknown sections and
planned structural changes (e.g. "agenda card 6 and topic block 6 will be removed —
or do you have a 6th topic?"); hidden slides that will be skipped (mention once);
then start at slide 1. Without a markdown, do not mention or ask for one.

Walk from slide 1 to the last slide **in deck order**. Work on ONE slide at a time
(at most two consecutive very small slides together). Never jump ahead to a later
slide. For the current slide show:

```
Slide 3 of 24 — TODAY'S CO-HOST · [Co-host name]
Progress: slide 3/24 · 41 placeholders left
  Auto-filled: footer "Nov 2026"
  Proposed: [Job title] → "Microsoft MVP Security, Contoso"  (source: mvp.microsoft.com/…)
  Still needed: [Co-host name], [Badge logo], [Website 1]
  Options: confirm · change · hide slide · delete slide · skip for now
```

0. Anything still unknown on this slide → ask here, in chat; never search the
   user's mail, calendar, Teams, files or directory for it.
1. Accept free-form answers (pasted text, links, lists) and map them to the
   placeholders; confirm the mapping when ambiguous. If an answer is (or contains)
   a CloudHour monthly markdown, handle it per Step 1.7.
2. On confirm/change: write the slide text **and** its speaker notes in one
   EditArtifact batch, re-scan, update the progress line, then move to the next
   slide. Hide / delete: apply, re-scan, continue.
3. **Skip for now** → record it; skipped slides are revisited in a second
   chronological pass after the last slide.
4. **Hidden slides** are skipped in the walk (named once in the opening message).
5. **Markdown TBD/empty entries** (only when a markdown was provided): when the walk
   reaches their card/slide/block, ask once: drop (default) or hide. Never write
   "TBD" or leave the placeholder.
6. **Structural changes** are decided when the walk reaches the **agenda slide**
   (or immediately, if a markdown arrives after the agenda slide was passed):
   - Fewer topics than blocks → remove unused agenda cards and topic blocks.
   - More topics or a longer topic → `duplicate_slide` the topic detail +
     round-table pair (or the optional bullet summary / image / quiz slide).
   Apply immediately, renumber `TOPIC N` labels and agenda cards, update the topic
   count, **re-scan**, recompute M, and continue the walk in the new order. If
   hiding is not possible with the available edit ops, ask the user to hide the
   slide in PowerPoint, or offer deletion instead.
7. **Links slide** ("Links to share" / "Worth a bookmark"): **maximum 7 links,
   7 cards**. With fewer than 7 links, remove the unused cards bottom-up — the
   last card of the right column first, then the next-last — and keep the
   remaining cards' layout, positions and formatting unchanged (no re-flow or
   restyle). Show the planned card removal in the slide's proposal and apply it
   on confirm. Never add an 8th card. If more than 7 links are provided, list them
   and ask which 7 to keep before writing. A bare URL without a title → propose a
   short title from a public-web fetch of that URL.
8. **Images** (`[Badge logo]`, `[Image or screenshot]`): ask the user to attach the
   image, or propose a public image found via a public web search (e.g. the official MVP logo)
   and get confirmation. Insert at the placeholder box's position and size
   (`add_image` / `replace_image`), then `delete_shape` the box. No image → ask:
   delete the box or hide the slide.

### Step 7 — Second pass and stop
After the last slide, walk the "skip for now" slides (and earlier slides that got
new proposals from a mid-walk markdown) again **in slide order**, same format. Stop
when **no `[...]` remains on any visible slide or its notes**, or the user
explicitly chose to hide/delete the slides that still have them. Hidden slides with
placeholders do not block completion. Then remove the `[TEMPLATE STRUCTURE]` block
from the agenda notes.

### Step 8 — Final QA and delivery
1. GetArtifactModel(render=true) → RenderArtifact per slide → check for overflowing
   or badly wrapping text; shorten text that overflows (keep titles short).
2. Verify the cover date is the first Wednesday of the month (or the user's stated
   date), the links slide has at most 7 cards and no empty ones, and every footer reads `CloudHour · Microsoft EMS Community · Nov 2026`,
   topic numbering is consecutive, agenda count matches the topic blocks (and, if a
   markdown was provided, its filled topics), and no "TBD" appears anywhere.
3. Final scan must return zero placeholders on visible slides.

## Output
Final chat message (short):
- The deck, referenced by its file handle (`CloudHour - Nov 2026.pptx`)
- **Monthly markdown** (only if the user provided one): where it came from, its
  status if stated, and what was taken from it
- **Filled automatically**: date, footers, numbering, notes, …
- **Researched & approved**: co-host profile, topic content with sources
- **From the user**: answers, images, …
- **Omitted / hidden / deleted**: dropped entries, removed cards/blocks, hidden
  slides (and any hidden slides that still contain placeholders)

## Guardrails
- **Allowed sources only** (see "Allowed sources"): the user's input, public-web
  research on what they provided, and the template/working copy. Never search or
  read mail, calendar/meetings/transcripts, Teams, OneNote, other OneDrive/
  SharePoint documents, people/org directory, prior Copilot conversations, or any
  M365/enterprise search — not even as fallback. Missing info → ask in chat.
- **Session date = first Wednesday of the month**, auto-filled without asking;
  only a user-stated date overrides it.
- **Links slide: at most 7 links/cards** — remove unused cards bottom-up (right
  column's last card first), never add cards; >7 links → ask which 7 to keep.
- **Never modify the original template** — every edit targets the dated copy's handle.
- **The monthly markdown is user-provided only** — never guess or derive its URL,
  never probe a repository, never ask for it unprompted.
- **Never fabricate** dates, deadlines, Message Center IDs, prices, names or URLs —
  take them from the user, a provided markdown or public-web research, or ask; only URLs that
  appeared in those sources go into slides and notes.
- Never write "TBD" or an unfilled markdown entry into the deck.
- Strict slide order: never jump ahead; one slide (at most two tiny ones) per step;
  a mid-walk markdown resumes at the current slide.
- Show proposals before writing; destructive changes (delete slide, delete shape,
  remove block) only after the user chose them.
- Keep the template's formatting: `set_text` with `keepFormat: true`, one set_text
  per shape per batch; never restyle.
- English only for slide content, notes and questions.
- If a live edit batch fails midway, re-read the model before retrying.
- If a provided markdown link cannot be fetched, say so, ask the user to paste the
  content, and continue the walk without it in the meantime.
