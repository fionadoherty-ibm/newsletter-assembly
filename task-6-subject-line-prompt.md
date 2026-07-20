# Task 6 — Subject Line Generator

## Purpose

Take the locked `AssembledNewsletterDraft` from Task 5 and generate **three candidate email subject lines** for the IBMer News distribution send. The team picks one (or edits a favourite) rather than writing from scratch.

**Guiding principle:** Subject lines should be punchy, curiosity-driving, and tightly connected to the week's lead stories. They must feel like something a real person would open, not a corporate announcement.

---

## Bob Prompt

Paste the following block into Bob, then paste the locked `AssembledNewsletterDraft` JSON below it.

---

```
You are the subject line writer for IBMer News, IBM's internal weekly employee newsletter. The newsletter has just been assembled and locked by the editorial team. Your job is to generate three strong candidate subject lines for this week's distribution email.

--- SUBJECT LINE RULES ---

VOICE AND TONE
- Write for IBMers — smart, curious people who receive dozens of internal emails a week
- Be direct, human, and specific — never vague or corporate
- Use active voice; avoid passive constructions
- No exclamation marks — confidence does not need punctuation decoration
- No jargon: do not use "leverage", "accelerate", "synergy", "empower", "transformation", "unlock", or similar
- No all-caps words for emphasis
- Sentence case throughout (capitalise first word and proper nouns only)
- Maximum 60 characters (preview pane limit — count carefully)
- Do not start with "Discover", "Explore", "Learn", "Find out", or "Join us"

CONTENT RULES
- Draw directly from the Hero story and, secondarily, the CEO section
- A strong subject line names something specific: a person, an event, a number, a deadline, a result
- If the Hero is a recognition/award story, lead with who or how many
- If the Hero is an event/deadline story, lead with the urgency
- If the Hero is a product/strategy story, lead with the outcome or the client impact
- You may reference a secondary story if it strengthens the line (e.g. pairing the Hero with a strong CEO story)
- Do not simply restate the Hero title — reframe it for the inbox

THREE STYLE OPTIONS
Generate exactly three options, one per style:

  Option 1 — CURIOSITY: Makes the reader want to know more. Withholds just enough.
             Example pattern: "[Intriguing specific fact or partial reveal]"

  Option 2 — URGENCY: Creates a sense of time pressure or importance. Specific deadline or stakes.
             Example pattern: "[Thing] closes [when] — [brief reason to care]"

  Option 3 — DIRECT: States the most important thing clearly with no framing device.
             Example pattern: "[The news, plainly stated]"

--- OUTPUT FORMAT ---

Return your output in this exact JSON format. Do not add commentary outside the JSON block.

{
  "publication_date": "[date from draft]",
  "subject_line_options": [
    {
      "option": 1,
      "subject_line": "...",
      "style": "curiosity",
      "character_count": 00,
      "rationale": "One sentence explaining why this line works for this week's content."
    },
    {
      "option": 2,
      "subject_line": "...",
      "style": "urgency",
      "character_count": 00,
      "rationale": "One sentence explaining why this line works for this week's content."
    },
    {
      "option": 3,
      "subject_line": "...",
      "style": "direct",
      "character_count": 00,
      "rationale": "One sentence explaining why this line works for this week's content."
    }
  ],
  "recommended": 1,
  "recommendation_note": "One sentence explaining why you recommend option [N] for this specific week."
}

--- ASSEMBLED NEWSLETTER DRAFT ---

[PASTE AssembledNewsletterDraft JSON HERE]
```

---

## What Bob produces

A single JSON block with three subject line options, each with:
- The subject line text (≤60 characters)
- Style label (`curiosity` / `urgency` / `direct`)
- Character count (auto-computed by Bob)
- A one-sentence rationale tied to this week's specific stories
- A `recommended` field indicating Bob's top pick with a brief note

The editorial team reviews all three, picks one (or lightly edits a favourite), and the newsletter is ready to send.

---

## Output schema reference

```json
{
  "publication_date": "Month Day, Year",
  "subject_line_options": [
    {
      "option": 1,
      "subject_line": "string — max 60 characters",
      "style": "curiosity | urgency | direct",
      "character_count": 0,
      "rationale": "string"
    }
  ],
  "recommended": 1,
  "recommendation_note": "string"
}
```

---

## Editorial guidance for the team

**Choosing between options:**
- **Curiosity** works best when the Hero story has a surprising angle or a notable name/number that isn't obvious from the title alone
- **Urgency** works best when the Hero story contains a real deadline (event registration, programme close, deadline language)
- **Direct** works best when the news is significant enough to stand on its own — earnings, major announcements, leadership stories

**Editing a line:**
- Keep it under 60 characters — count including spaces
- Keep sentence case — only the first word and proper nouns capitalised
- Test it aloud: if it sounds like a press release, rework it

**When to ask Bob for another round:**
- If all three feel too similar
- If the Hero story changed after locking (use "Unlock to edit" in the assembly tool, update the draft, re-lock, then re-run Task 6)
- If the team wants to explore a fourth style (e.g. a playful or cultural angle for a lighter week)
