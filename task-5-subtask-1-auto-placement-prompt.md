# Task 5 / Sub-Task 1 — Auto-Placement Prompt
## Bob Prompt: Newsletter Assembly — Auto-Place Stories into Sections

---

## HOW TO USE THIS PROMPT

Paste this prompt into your Bob session. Then paste all your Task 3 and Task 4 outputs below it (as JSON arrays — see Input Format below). Bob will produce a single `AssembledNewsletterDraft` JSON object, ready to be passed into the Sub-Task 2 HTML assembly artifact.

---

## THE PROMPT

---

You are assembling the weekly IBMer News newsletter. You have received a set of approved story objects from the editorial pipeline. Your job is to auto-place every story into the correct newsletter section and produce a single structured JSON draft ready for the editor to review and reorder.

### REFERENCE: EDITORIAL PATTERN GUIDE

Before placing any stories, read the attached editorial pattern guide (`IBMer News editorial pattern guidance.docx`). It documents how 16 real IBMer News newsletters were structured — which stories earned the Hero slot, how the CEO slot was consistently used, how sections were differentiated, and how the team handled quiet weeks with no obvious Hero candidate.

Key rules to apply from the pattern guide:

**Hero slot — use this priority order:**
1. Tier 1 (always Hero if present): earnings results, annual/quarterly report, CEO year-start message, major acquisition closing, external press validation
2. Tier 2 (Hero when no Tier 1 exists): flagship event recap (Think, Kickoff, WEF, Davos), major product launch with headline metrics, IBMer-built internal tool launch, top-tier award recognition (Golden Circle, IBM Fellows, Turing Award)
3. If neither Tier 1 nor Tier 2 exists, promote in this order: (1) people/recognition story, (2) internal tool or program launch, (3) thought-leadership explainer, (4) mandatory compliance item for urgency — and mark as `"forced_hero": true`

**CEO slot — use this priority order:**
1. CEO Office Hours recap or replay link (~70% of issues)
2. Arvind at an external event (WEF, World Governments Summit, podcast, TV appearance)
3. CEO message or letter to IBMers
4. Leadership team change announcement
- Detect CEO content by scanning `final_title` and `final_abstract` for: `CEO`, `Arvind`, `Office Hours`, `leadership`, `year-start`
- If Office Hours has not yet aired, look for teaser language with a date and "Register now" CTA

**Section differentiation — when a story could fit multiple sections:**
- **In the News:** product explainer, client win, IBV research, industry partnership, awards/analyst recognition — 2–4 stories, every issue
- **Latest Updates:** compliance deadlines, IT tool transitions, security advisories — usually 1 item, ~60% of issues
- **Strategy Spotlight:** deep technical feature needing more space than In the News allows (quantum, Consulting Advantage, AI explainer)
- **Featured Stories / Smarter business in action:** named client result with specific metrics
- **Around IBM:** IBMer community story, office opening, CSR/Week of Impact, employee spotlight
- **Just for #fun:** always the last editorial block — crossword clue, LEGO, video, sports tie-in, SXSW activation

**Sports partnership stories:** treat as Hero if it's an announcement or launch; treat as In the News if it's ongoing coverage.

When making a placement decision for an ambiguous story, ask: which section carried this type of content in the pattern guide? Use that as the tiebreaker.

### INPUTS YOU WILL RECEIVE

You will receive two JSON arrays:

1. **Editorial Review Reports** — one per story, from Task 3. Each object contains:
   - `story_id` — unique identifier
   - `approved` — true | false (only process stories where `approved: true`)
   - `final_title` — the approved headline
   - `final_abstract` — the approved abstract
   - `final_cta` — the approved call to action
   - `suggested_category` — the section this story was originally tagged for: `In the News` | `Featured Stories` | `Strategy Spotlight` | `Latest Updates` | `Around IBM` | `Just for #fun`
   - `source_type` — `w3` | `press release` | `blog` | `other`
   - `flags` — array of any unresolved editorial issues (may be empty)

2. **Image Recommendation Blocks** — one per story, from Task 4. Each object contains:
   - `story_id` — matches the Editorial Review Report
   - `requires_image` — true | false
   - `image_status` — `found` | `not found` | `needs manual sourcing`
   - `recommended_sources` — array of image candidates (url_or_path, description)

### SCORING RULES

Before placing stories, calculate an **editorial score** for each story using these rules. Add points for every criterion that applies:

| Signal | Points |
|---|---|
| Title or abstract mentions recognition, award, congratulations, honour, or celebrates colleagues | +10 |
| Title or abstract contains the keyword `CEO` | +8 |
| Title or abstract contains explicit deadline or time-sensitive language — must include a specific date, day, or phrase such as "register by [date]", "closes [day]", "deadline [date]", "last chance", "ends Friday", "this Friday" — do NOT award this for vague future language like "what's ahead" or "coming soon" | +7 |
| Title or abstract mentions Think conference | +7 |
| Title or abstract mentions AI, watsonx, or artificial intelligence | +6 |
| Title or abstract mentions strategy, transformation, or innovation | +6 |
| Title or abstract mentions client success, client win, or customer | +5 |
| Content is directly actionable for most IBMers (e.g. HR update, benefits, policy) | +5 |
| Title or abstract mentions volunteering, community, or Week of Impact | +3 |
| `suggested_category` is `Just for #fun` or content is clearly light/cultural | +2 |

Record the total as `editorial_score` on each story object.

### SECTION ASSIGNMENT RULES

Apply these rules in order. Once a story is assigned to a section, do not re-assign it.

**Step 1 — CEO slot (multi-story)**
- Find ALL stories where `final_title` or `final_abstract` contains any of these keywords (case-insensitive): `CEO`, `Arvind`, `Office Hours`, `leadership`, `year-start`
- Assign all matching stories to the `CEO` section. Sort them by `editorial_score` descending, using the CEO content priority order as a tiebreaker: Office Hours replay first, then external event appearances, then CEO messages, then leadership change announcements.
- If no stories match any CEO keyword, leave the CEO section empty and set `"ceo_slot_empty": true`.

**Step 2 — Hero slot (single story)**
- From the remaining unassigned stories (excluding the CEO story), apply the Hero tier priority from the pattern guide:
  1. **Tier 1** (always Hero if present): earnings results, annual/quarterly report, CEO year-start message, major acquisition closing, external press validation — assign immediately, no score check needed
  2. **Tier 2** (Hero when no Tier 1 exists): flagship event recap (Think, Kickoff, WEF, Davos), major product launch with headline metrics, IBMer-built internal tool launch, top-tier award recognition (Golden Circle, IBM Fellows, Turing Award)
  3. **No Tier 1 or Tier 2:** check `editorial_score >= 25` and select the highest scorer; if none qualify, use this forced-hero promotion order: (1) people/recognition story, (2) internal tool or program launch, (3) thought-leadership explainer, (4) mandatory compliance item
- Mark the selected story `"forced_hero": true` if it was selected under Tier 2 or lower without clearing the 25-point threshold.
- Assign it to the `Hero` section.

**Step 3 — Remaining sections**
- Assign all remaining unassigned stories to their `suggested_category` section directly:
  - `In the News` → In the News section
  - `Featured Stories` → Featured Stories section
  - `Strategy Spotlight` → Strategy Spotlight section
  - `Latest Updates` → Latest Updates section
  - `Around IBM` → Around IBM section
  - `Just for #fun` → Just for #fun section

**Step 4 — Order within sections**
- Within each section, sort stories by `editorial_score` descending (highest score = order 1).
- Assign an integer `order` field starting at 1.

**Step 5 — Merge image data**
- For each story, look up its matching `ImageRecommendationBlock` by `story_id`.
- Attach:
  - `image_url`: the first `url_or_path` from `recommended_sources` if `image_status` is `found`; otherwise `null`
  - `image_status`: carry across as-is (`found` | `not found` | `needs manual sourcing`)

**Step 6 — Skip unapproved stories**
- Any story with `approved: false` must NOT be placed in the newsletter. Collect their `story_id` values in a separate `skipped_stories` array in the output.

### OUTPUT FORMAT

**Critical output rules — read carefully before generating:**
- Output a **single root JSON object** — do NOT wrap it in a `newsletter_draft`, `meta`, or any other outer key.
- `sections` must be a **JSON array** (an ordered list `[...]`), NOT an object keyed by section name.
- Every section object must include all three fields: `section_name`, `single_slot`, and `stories`.
- The top-level object must include exactly these four fields at the root: `publication_date`, `headline`, `ceo_slot_empty`, `skipped_stories`, and `sections`.
- `skipped_stories` must be a flat array of `story_id` strings only — for example `["S006"]` — do NOT nest objects inside it.
- Do NOT add `_note`, `meta`, `generated_at`, or any extra fields anywhere in the output.
- Do NOT add any commentary, explanation, or text outside the JSON block — output the JSON first, then the Assembly Notes section.

Produce the JSON object in exactly this structure:

```json
{
  "publication_date": "PROVIDE THE DATE OR LEAVE AS: TBC",
  "headline": "IBMer News This Week. The news you need to know.",
  "ceo_slot_empty": false,
  "skipped_stories": [],
  "sections": [
    {
      "section_name": "Hero",
      "single_slot": true,
      "stories": [
        {
          "story_id": "...",
          "final_title": "...",
          "final_abstract": "...",
          "final_cta": "...",
          "source_type": "...",
          "suggested_category": "...",
          "editorial_score": 0,
          "forced_hero": false,
          "requires_image": true,
          "image_url": "... or null",
          "image_status": "found | not found | needs manual sourcing",
          "flags": [],
          "order": 1
        }
      ]
    },
    {
      "section_name": "CEO",
      "single_slot": true,
      "stories": []
    },
    {
      "section_name": "In the News",
      "single_slot": false,
      "stories": []
    },
    {
      "section_name": "Featured Stories",
      "single_slot": false,
      "stories": []
    },
    {
      "section_name": "Strategy Spotlight",
      "single_slot": false,
      "stories": []
    },
    {
      "section_name": "Latest Updates",
      "single_slot": false,
      "stories": []
    },
    {
      "section_name": "Around IBM",
      "single_slot": false,
      "stories": []
    },
    {
      "section_name": "Just for #fun",
      "single_slot": false,
      "stories": []
    }
  ]
}
```

Sections with no stories should still appear in the output with an empty `stories` array. Preserve the section order exactly as shown above.

After the JSON block, add a brief plain-English **Assembly Notes** section (3–5 bullet points) summarising:
- Which story was assigned to Hero and why
- Which story was assigned to CEO and why (or that the CEO slot is empty)
- Whether any stories were force-promoted to Hero below the 25-point threshold
- Any sections that are empty
- Any stories that were skipped (not approved)

---

## INPUT FORMAT

Paste your Task 3 and Task 4 outputs below this line, in this format:

```
EDITORIAL_REVIEW_REPORTS:
[ paste Task 3 JSON array here ]

IMAGE_RECOMMENDATION_BLOCKS:
[ paste Task 4 JSON array here ]
```

---

## SAMPLE INPUT (for testing)

Use this sample input to test the prompt before connecting live Task 3 and Task 4 outputs.

```
EDITORIAL_REVIEW_REPORTS:
[
  {
    "story_id": "S001",
    "approved": true,
    "final_title": "Congratulations to our IBM Golden Circle invitees!",
    "final_abstract": "This year's Golden Circle honourees represent the best of IBM — top performers who have gone above and beyond for clients and colleagues. See if a colleague you know made the list.",
    "final_cta": "Celebrate our colleagues",
    "suggested_category": "In the News",
    "source_type": "w3",
    "flags": []
  },
  {
    "story_id": "S002",
    "approved": true,
    "final_title": "CEO Office Hours replay: watch Arvind's latest update",
    "final_abstract": "Missed this week's CEO Office Hours? Catch the full replay to hear Arvind's perspective on IBM's AI strategy, client momentum, and what's ahead for the rest of the year.",
    "final_cta": "Watch the replay",
    "suggested_category": "Featured Stories",
    "source_type": "w3",
    "flags": []
  },
  {
    "story_id": "S003",
    "approved": true,
    "final_title": "Register now for Think 2026 — deadline this Friday",
    "final_abstract": "Think 2026 is IBM's flagship conference and your chance to connect with clients, partners, and colleagues from across the company. Early-bird registration closes this Friday — don't miss your spot.",
    "final_cta": "Register now",
    "suggested_category": "In the News",
    "source_type": "w3",
    "flags": []
  },
  {
    "story_id": "S004",
    "approved": true,
    "final_title": "How watsonx is transforming client operations at scale",
    "final_abstract": "Three clients share how IBM watsonx has helped them automate workflows, cut costs, and accelerate decision-making. Real results, real impact.",
    "final_cta": "Read the stories",
    "suggested_category": "Strategy Spotlight",
    "source_type": "blog",
    "flags": []
  },
  {
    "story_id": "S005",
    "approved": true,
    "final_title": "IBM Week of Impact is coming — here's how to get involved",
    "final_abstract": "IBM Week of Impact kicks off next month. Find volunteering opportunities near you, sign up for virtual events, and make a difference in your community.",
    "final_cta": "Find opportunities",
    "suggested_category": "Around IBM",
    "source_type": "w3",
    "flags": []
  },
  {
    "story_id": "S006",
    "approved": false,
    "final_title": "Q2 internal operations update",
    "final_abstract": "Draft pending final review.",
    "final_cta": "Read more",
    "suggested_category": "Latest Updates",
    "source_type": "other",
    "flags": [{ "field": "final_abstract", "issue": "Too vague", "rule_violated": "smart brevity", "suggestion": "Add specific details" }]
  },
  {
    "story_id": "S007",
    "approved": true,
    "final_title": "18 across: maker of the world's largest quantum computer?",
    "final_abstract": "This week's fun fact: IBM holds the record for the world's largest quantum computer. How many qubits? Find out inside.",
    "final_cta": "Get the answer",
    "suggested_category": "Just for #fun",
    "source_type": "other",
    "flags": []
  }
]

IMAGE_RECOMMENDATION_BLOCKS:
[
  { "story_id": "S001", "requires_image": true, "image_status": "found", "recommended_sources": [{ "source_type": "story webpage", "url_or_path": "https://w3.ibm.com/golden-circle-2026/hero.jpg", "description": "Golden Circle banner image" }] },
  { "story_id": "S002", "requires_image": false, "image_status": "not found", "recommended_sources": [] },
  { "story_id": "S003", "requires_image": true, "image_status": "found", "recommended_sources": [{ "source_type": "DAM", "url_or_path": "/dam/think-2026/think-banner.jpg", "description": "Think 2026 event banner" }] },
  { "story_id": "S004", "requires_image": false, "image_status": "not found", "recommended_sources": [] },
  { "story_id": "S005", "requires_image": true, "image_status": "needs manual sourcing", "recommended_sources": [] },
  { "story_id": "S006", "requires_image": false, "image_status": "not found", "recommended_sources": [] },
  { "story_id": "S007", "requires_image": false, "image_status": "not found", "recommended_sources": [] }
]
```

---

## EXPECTED SAMPLE OUTPUT (for validation)

When run against the sample input above, Bob should produce an `AssembledNewsletterDraft` where:
- **Hero** → S001 (Congratulations / Golden Circle — recognition +10, score likely highest at 10; no story exceeds 25 so `forced_hero: true`)
- **CEO** → S002 (contains keyword `CEO` in title — "CEO Office Hours")
- **In the News** → S003 (Think deadline — originally tagged In the News; Hero is taken by S001)
- **Strategy Spotlight** → S004 (watsonx / AI story)
- **Around IBM** → S005 (volunteering)
- **Just for #fun** → S007 (fun/culture)
- **Featured Stories** → empty
- **Latest Updates** → empty
- **Skipped** → S006 (approved: false)
- Assembly Notes should mention forced hero promotion and the empty Featured Stories / Latest Updates sections
