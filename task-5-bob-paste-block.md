# READY-TO-PASTE BOB TEST BLOCK
# Copy everything below the line and paste it into Bob in one go.
# ─────────────────────────────────────────────────────────────────────────────

---

EDITORIAL PATTERN GUIDE:

IBMer News — Editorial Pattern Guide

Structure

Fixed slots (every issue): Hero (top/lead story) · CEO slot · "In the news this week" section · Personalized news callout · Fun/light closer

Recurring optional slots: "The latest updates" (compliance/IT deadlines) · "Around IBM" (community/culture) · "Spotlight" (deep technical feature) · "Think ahead" / event promo · "Smarter business in action" (client win)

Footer (always): Contact/Slack link · past newsletters · privacy policy · subscription center

Hero Slot — What earns the top position

Tier 1 triggers (always Hero): Earnings results · Annual/Quarterly report · CEO year-start message · Major acquisition closing (e.g., Confluent) · External press validation (e.g., The Economist IBM profile)

Tier 2 triggers (Hero when no Tier 1 exists): Flagship event recap (Think, Kickoff, WEF, Davos) · Major product launch with headline metrics (FlashSystem.ai, quantum milestone) · IBMer-built internal tool launch (watsonx Workshop) · Top-tier award recognition (Golden Circle, IBM Fellows, Turing Award)

Hero format: 2–4 sentence summary + pull quote or stat + single CTA ("Read more" / "Watch now"). No bullet lists in the Hero block.

Why stories earn Hero: Company-wide relevance, time-sensitivity, external credibility signal, or direct employee impact (pay, benefits, tools).

CEO Slot — Patterns

Always present (one exception: Jan 9, 2025, where it merges with the Hero).

Content types in priority order:
1. CEO Office Hours recap/replay link (most common; appears ~70% of issues)
2. Arvind at an external event (WEF, World Governments Summit, podcast, Bloomberg TV)
3. CEO message or letter to IBMers (year-start, earnings, memorial)
4. Leadership team change announcement (promotions, retirements)

Tone: Always 2–4 sentences, first-person quote or paraphrase of Arvind, one CTA.

When Office Hours is upcoming (not yet aired): teaser with date, channel (#join-arvind), and "Register now" CTA instead of replay.

Recurring Sections & Story Types

In the news this week — Product explainer, client win, IBV research, industry partnership, awards/analyst recognition. Every issue; 2–4 stories.
The latest updates — Compliance deadlines (Core Training), IT tool transitions (SAP SuccessFactors, passkeys, Box/OneDrive), security advisories. ~60% of issues; usually 1 item.
Spotlight — Deep technical feature (quantum, Consulting Advantage); longer format. Occasional; used when a story needs more space than "In the news" allows.
Think ahead / event promo — Client nomination CTAs, IBMer registration, countdown language. Appears heavily in Q1–Q2 build-up to Think; ~40% of issues.
Smarter business in action — Named client result with 2–3 bullet metrics. ~30% of issues; often tied to a product story in the same issue.
Around IBM — IBMer community story, office opening, CSR/Week of Impact, employee spotlight. ~50% of issues.
Just for #fun — Crossword clue, LEGO build, video trick, SXSW activation, sports tie-in. ~70% of issues; always last editorial block before footer.

Sports Partnership Tie-ins: Wimbledon, US Open, UFC, Scuderia Ferrari, Masters, Grammys, lacrosse each appear. Treatment: Hero if announcement/launch; In the News if ongoing. Always includes a social share CTA.

Weeks with No Obvious Hero-Quality Story — promotion order: (1) people/recognition story, (2) internal tool or program launch, (3) thought-leadership explainer, (4) mandatory compliance item for urgency.

---

ASSEMBLY PROMPT:

You are assembling the weekly IBMer News newsletter. You have received a set of approved story objects from the editorial pipeline. Your job is to auto-place every story into the correct newsletter section and produce a single structured JSON draft ready for the editor to review and reorder.

Use the Editorial Pattern Guide above as your primary reference for all placement decisions.

Key rules to apply:

Hero slot — use this priority order:
1. Tier 1 (always Hero if present): earnings results, annual/quarterly report, CEO year-start message, major acquisition closing, external press validation — assign immediately, no score check needed
2. Tier 2 (Hero when no Tier 1 exists): flagship event recap (Think, Kickoff, WEF, Davos), major product launch with headline metrics, IBMer-built internal tool launch, top-tier award recognition (Golden Circle, IBM Fellows, Turing Award)
3. If neither Tier 1 nor Tier 2 exists, promote in this order: (1) people/recognition story, (2) internal tool or program launch, (3) thought-leadership explainer, (4) mandatory compliance item — and mark as "forced_hero": true

CEO slot — use this priority order:
1. CEO Office Hours recap or replay link
2. Arvind at an external event
3. CEO message or letter to IBMers
4. Leadership team change announcement
Detect CEO content by scanning final_title and final_abstract for: CEO, Arvind, Office Hours, leadership, year-start

Section differentiation — when a story could fit multiple sections:
- In the News: product explainer, client win, IBV research, industry partnership, awards/analyst recognition
- Latest Updates: compliance deadlines, IT tool transitions, security advisories
- Strategy Spotlight: deep technical feature needing more space (quantum, AI explainer, Consulting Advantage)
- Featured Stories: named client result with specific metrics
- Around IBM: IBMer community story, CSR/Week of Impact, employee spotlight
- Just for #fun: always last — crossword, LEGO, video, sports tie-in

SCORING RULES

Calculate an editorial score for each story. Add points for every criterion that applies:

- Title or abstract mentions recognition, award, congratulations, honor, or celebrates colleagues: +10
- Title or abstract contains CEO: +8
- Title or abstract contains explicit deadline language with a specific date or day (e.g. "deadline this Friday", "closes [date]", "last chance", "ends Friday") — do NOT award for vague future language like "what's ahead" or "coming soon": +7
- Title or abstract mentions Think conference: +7
- Title or abstract mentions AI, watsonx, or artificial intelligence: +6
- Title or abstract mentions strategy, transformation, or innovation: +6
- Title or abstract mentions client success, client win, or customer: +5
- Content is directly actionable for most IBMers (HR update, benefits, policy): +5
- Title or abstract mentions volunteering, community, or Week of Impact: +3
- suggested_category is "Just for #fun" or content is clearly light/cultural: +2

SECTION ASSIGNMENT RULES

Apply in order. Once assigned, do not re-assign.

Step 1 — CEO slot (single story)
Find all stories where final_title or final_abstract contains any of: CEO, Arvind, Office Hours, leadership, year-start (case-insensitive). Select the highest scorer. If tied, use CEO priority order above. If no match, set ceo_slot_empty: true.

Step 2 — Hero slot (single story)
From remaining unassigned stories, apply Tier 1 then Tier 2 priority. If neither applies, check editorial_score >= 25 and take the highest. If none qualify, use forced-hero promotion order above. Mark forced_hero: true if promoted without reaching Tier 1 or Tier 2 or scoring >= 25.

Step 3 — Remaining sections
Assign all remaining stories to their suggested_category section.

Step 4 — Order within sections
Sort by editorial_score descending. Assign order starting at 1.

Step 5 — Merge image data
For each story, attach image_url (first url_or_path if image_status is "found", otherwise null) and image_status from the matching ImageRecommendationBlock.

Step 6 — Skip unapproved stories
Any story with approved: false must NOT be placed. Collect story_id values in skipped_stories.

OUTPUT FORMAT — CRITICAL RULES:
- Output a single root JSON object — do NOT wrap in newsletter_draft, meta, or any outer key
- sections must be a JSON array [...], NOT an object keyed by section name
- Every section must include: section_name, single_slot, and stories
- Top-level fields must be exactly: publication_date, headline, ceo_slot_empty, skipped_stories, sections
- skipped_stories must be a flat array of story_id strings only e.g. ["S006"] — no nested objects
- Do NOT add _note, meta, generated_at, or any extra fields
- Output the JSON block first, then Assembly Notes — no other text outside these two

Produce the JSON in exactly this structure:

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
    { "section_name": "CEO", "single_slot": true, "stories": [] },
    { "section_name": "In the News", "single_slot": false, "stories": [] },
    { "section_name": "Featured Stories", "single_slot": false, "stories": [] },
    { "section_name": "Strategy Spotlight", "single_slot": false, "stories": [] },
    { "section_name": "Latest Updates", "single_slot": false, "stories": [] },
    { "section_name": "Around IBM", "single_slot": false, "stories": [] },
    { "section_name": "Just for #fun", "single_slot": false, "stories": [] }
  ]
}

Sections with no stories must still appear with an empty stories array. Preserve section order exactly as shown.

After the JSON block, add a brief plain-English Assembly Notes section (3–5 bullet points) covering:
- Which story was assigned to Hero and why
- Which story was assigned to CEO and why (or that the slot is empty)
- Whether any stories were force-promoted to Hero
- Any empty sections
- Any skipped stories

---

EDITORIAL_REVIEW_REPORTS:
[
  {
    "story_id": "S001",
    "approved": true,
    "final_title": "Congratulations to our IBM Golden Circle invitees!",
    "final_abstract": "This year's Golden Circle honorees represent the best of IBM — top performers who have gone above and beyond for clients and colleagues. See if a colleague you know made the list.",
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
