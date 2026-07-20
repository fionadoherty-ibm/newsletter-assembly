# Task 5 — Newsletter Assembly and Story Ordering
## Plan

**Status:** [x] done
**Last updated:** June 2025

---

## Overview

Task 5 takes all approved Editorial Review Reports (Task 3) and Image Recommendation Blocks (Task 4) and produces a fully assembled, editor-reviewed newsletter draft. 

Bob auto-places stories into sections using the `suggested_category` and editorial score from the upstream data. Editors then interact with an **HTML artifact** rendered directly in the Bob chat UI — they can reorder stories within sections, move stories between sections, and reassign the Hero and CEO slots freely. When the editor is satisfied, they click **Lock**, which posts the final structured order back into the Bob chat thread so Task 6 (Subject Line Generator) can proceed automatically.

**Guiding principle:** Bob suggests. Editors own the story.

---

## Inputs

- All approved `EditorialReviewReport` objects from Task 3 (one per story)
- All `ImageRecommendationBlock` objects from Task 4 (one per story)

## Output

A single `AssembledNewsletterDraft` JSON object:

```json
{
  "publication_date": "Month Day, Year",
  "headline": "IBMer News This Week. The news you need to know.",
  "sections": [
    {
      "section_name": "Hero | CEO | In the News | Featured Stories | Strategy Spotlight | Latest Updates | Around IBM | Just for #fun",
      "stories": [
        {
          "story_id": "...",
          "final_title": "...",
          "final_abstract": "...",
          "final_cta": "...",
          "image": "url or DAM path | none",
          "order": 1
        }
      ]
    }
  ]
}
```

---

## Sub-Tasks

---

### Sub-Task 1 — Auto-placement prompt

**Intent:**  
Build the Bob prompt that receives all Task 3 and Task 4 outputs and automatically assigns each story to its initial section and rank order, ready for the HTML assembly view.

**Expected Outcomes:**
- Each story is placed into one of the 8 newsletter sections based on its `suggested_category`
- Within each section, stories are ranked by their editorial score (descending)
- The Hero slot receives the story with the highest score that meets the 25+ threshold; if no story scores 25+, the highest-scoring story is auto-promoted and marked as a forced choice
- The CEO slot receives the story whose title, abstract, or a tag contains the keyword `CEO`; if multiple stories match, the highest scorer is selected
- The prompt outputs a single structured data object (the initial `AssembledNewsletterDraft`) ready to be passed to the HTML artifact

**Todo List:**
1. Write a prompt that accepts a batch of Editorial Review Reports and Image Recommendation Blocks
2. Define the section mapping rule: `suggested_category` → section name (direct mapping for all 6 standard categories; Hero and CEO assigned by score logic)
3. Implement Hero assignment: find stories with `editorial_score >= 25`; take the highest scorer; if none qualify, take the highest overall and flag it as `"forced_hero": true`
4. Implement CEO assignment: scan `final_title`, `final_abstract`, and any tag fields for the keyword `CEO`; assign the highest-scoring match to the CEO slot
5. Sort remaining stories within each section by `editorial_score` descending
6. Merge image data: for each story, attach `image_url` or `image_status` from the matching Image Recommendation Block
7. Output the initial `AssembledNewsletterDraft` JSON object

**Relevant Context:**
- Section names and order: Hero, CEO, In the News, Featured Stories, Strategy Spotlight, Latest Updates, Around IBM, Just for #fun
- Editorial scoring model in `IBM Bob Newsletter Guidance.docx` (recognition +10, CEO +8, deadline +7, Think +7, AI/watsonx +6, strategy +6, client success +5, employee utility +5, volunteerism +3, fun +2)
- Hero threshold: score > 25
- `suggested_category` values from Task 1: In the News | Featured Stories | Strategy Spotlight | Latest Updates | Around IBM | Just for #fun
- Image requirement: Hero, CEO, and Around IBM / Just for #fun sections always require a visual

**Status:** [x] done — see `task-5-subtask-1-auto-placement-prompt.md`

---

### Sub-Task 2 — HTML assembly artifact

**Intent:**  
Build the interactive HTML artifact that renders all auto-placed stories in their sections and lets editors reorder and reprioritize freely before locking the draft.

**Expected Outcomes:**
- Editors see all 8 newsletter sections laid out vertically in their canonical order
- Each story is displayed as a card showing: final title, truncated abstract (first ~120 characters with an expand toggle for the full text), CTA, image status indicator (green dot = image confirmed / red dot = image missing), and source type label (w3 / press release / blog / other)
- Stories flagged as `"forced_hero": true` display a visible warning badge
- Editors can drag and drop stories to reorder within a section
- Editors can drag stories between sections (including into and out of Hero and CEO)
- Hero and CEO sections are visually distinguished as single-story slots; if an editor drops a second story onto Hero or CEO, the displaced story moves to its `suggested_category` section automatically
- A "Lock Newsletter" button is present and only enabled when both the Hero and CEO slots contain exactly one story each
- Sections with no stories are shown as empty drop zones (editors can drag stories into them)

**Todo List:**
1. Design the overall page layout: fixed header with publication date and newsletter headline, then sections stacked vertically
2. Design the story card component: title, abstract truncated to ~120 characters with an inline "Show more / Show less" toggle for the full text, CTA in a pill/badge, image status dot (green/red), source type label, drag handle
3. Add a warning badge to the forced-hero card
4. Implement drag-and-drop using the HTML5 Drag and Drop API (no external library dependency, keeping the artifact self-contained)
5. Implement within-section reordering
6. Implement cross-section movement (drag from any section to any other section)
7. Implement the Hero/CEO single-slot displacement rule: when a card is dropped on a full single-slot section, the previously occupying card is moved to its `suggested_category` section
8. Implement the Lock button enabled/disabled state (requires Hero and CEO each to have exactly 1 story)
9. Apply IBM-aligned visual styling (dark background `#161616`, IBM Plex Sans font, Carbon Design System color tokens for badges and indicators where feasible within a self-contained artifact)
10. Test the artifact with a representative set of sample stories across all 8 sections

**Relevant Context:**
- This is a self-contained HTML artifact generated inside Bob chat — no external JS libraries, no server calls
- The artifact receives the initial `AssembledNewsletterDraft` JSON from Sub-Task 1 as its data source (injected directly into the artifact script)
- Section canonical order: Hero → CEO → In the News → Featured Stories → Strategy Spotlight → Latest Updates → Around IBM → Just for #fun
- Image requirement warnings are especially important for Hero, CEO, and Around IBM / Just for #fun

**Status:** [x] done — see `newsletter_assembly` HTML artifact (Sub-Tasks 2, 3, 4 combined in artifact)

---

### Sub-Task 3 — Lock and postMessage handoff

**Intent:**  
When the editor clicks "Lock Newsletter", the artifact captures the final ordered state and posts it back into the Bob chat thread as a structured message so that Task 6 can proceed without any manual copy-paste.

**Expected Outcomes:**
- Clicking "Lock Newsletter" serialises the current state of all sections and story order into an `AssembledNewsletterDraft` JSON object reflecting the editor's final arrangement
- The JSON is posted to the Bob chat thread via `window.parent.postMessage`
- The message payload is formatted so Bob can detect it and immediately use it as input to Task 6
- A confirmation state is shown inside the artifact after locking: "Newsletter locked. Bob is generating subject line options." — the drag handles and Lock button become disabled so the order cannot be changed after locking
- If an editor wants to unlock and make further changes, a secondary "Unlock to edit" button is available

**Todo List:**
1. Write the `handleLock()` function that walks the DOM (or an internal state object) to build the final `AssembledNewsletterDraft` JSON in the correct output format
2. Assign an `order` integer (1-based) to each story within its section based on its current rendered position
3. Call `window.parent.postMessage({ type: "NEWSLETTER_LOCKED", payload: assembledDraft }, "*")` 
4. Update the artifact UI to locked state: disable drag handles, disable Lock button, show confirmation message, show Unlock button
5. Write the Bob prompt handler that listens for the `NEWSLETTER_LOCKED` message and passes the payload automatically to Task 6
6. Implement Unlock: clicking "Unlock to edit" re-enables drag handles and the Lock button, clears the confirmation message
7. Test the full round-trip: auto-placement → editor reordering → Lock → Bob receives payload → Task 6 triggered

**Relevant Context:**
- `window.parent.postMessage` is the standard mechanism for iframe-to-parent communication in Bob HTML artifacts
- The payload must match the `AssembledNewsletterDraft` schema exactly as defined in the Task 5 output spec in `newsletter-draft-wizard-plan.txt`
- Task 6 input is the full `AssembledNewsletterDraft` object — no transformation needed if the schema is preserved

**Status:** [x] done — Lock, postMessage, and Unlock implemented in `newsletter_assembly` artifact

---

### Sub-Task 4 — Validation and pre-lock warnings

**Intent:**  
Before the editor locks the newsletter, surface any outstanding issues that should be resolved — missing images on required visual sections, empty required sections, unresolved editorial flags — without blocking the editor from locking if they choose to proceed anyway.

**Expected Outcomes:**
- A persistent validation panel (sidebar or footer bar) shows a live count of warnings as the editor reorders stories
- Warning types:
  - **Missing image** on Hero, CEO, or Around IBM / Just for #fun stories (red)
  - **Empty section** for Hero or CEO (red, also disables Lock button)
  - **Forced hero** promotion in effect — highest-scoring story auto-promoted below the threshold (amber)
  - **Unresolved editorial flags** — stories with `approved: false` from Task 3 that the editor placed into the newsletter (amber)
- Red warnings that affect Hero or CEO emptiness disable the Lock button
- All other warnings are advisory only — editor can acknowledge and proceed
- A "Review warnings" button in the validation panel scrolls to the relevant story card

**Todo List:**
1. Define the four warning types and their severity levels (red = blocking for Hero/CEO empty; amber = advisory)
2. Build a `validateDraft()` function that runs after every drag-and-drop action and on initial render
3. Render the validation panel as a sticky footer bar inside the artifact showing warning count and a breakdown by type
4. Implement the "Review warnings" scroll-to behaviour
5. Link the Lock button enabled state to the absence of red (blocking) warnings only
6. Ensure forced-hero warning card badge (from Sub-Task 2) and the validation panel are in sync

**Relevant Context:**
- Validation runs client-side only, inside the artifact — no additional Bob prompt calls needed
- `approved: false` flag is available on each story's `EditorialReviewReport` input data
- `image_status: "not found" | "needs manual sourcing"` is available on each story's `ImageRecommendationBlock`
- `forced_hero: true` is set in Sub-Task 1 output

**Status:** [x] done — validation panel with red/amber warnings and Lock gate implemented in `newsletter_assembly` artifact

---

## Handoff

| From | To | Data |
|------|----|------|
| Task 3 (all stories) | Task 5 Sub-Task 1 | `EditorialReviewReport[]` |
| Task 4 (all stories) | Task 5 Sub-Task 1 | `ImageRecommendationBlock[]` |
| Sub-Task 1 | Sub-Task 2 | Initial `AssembledNewsletterDraft` JSON (injected into artifact) |
| Sub-Task 2 + 3 | Task 6 | Final `AssembledNewsletterDraft` JSON (via postMessage → Bob) |

Sub-Tasks 2, 3, and 4 are each **separately testable deliverables** — built and validated independently before being combined into the final artifact.

---

## Constraints and Non-Goals

- The artifact must be self-contained HTML/CSS/JS — no external library dependencies
- This task does NOT re-run editorial validation — that is Task 3's responsibility
- This task does NOT generate subject lines — that is Task 6's responsibility
- Marketo export is explicitly out of scope for this build (noted as a future goal in the master plan)
- The tool must work for nontechnical users — UI language should be plain, actions obvious
