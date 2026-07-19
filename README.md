# IBMer News — Newsletter Assembly

**[▶ Open the live tool](https://fionadoherty-ibm.github.io/newsletter-assembly/)**

A browser-based tool for assembling and reviewing the IBMer News weekly newsletter before it goes to production.

## What it does

- Displays all approved stories for the current issue, auto-placed into sections (Hero, CEO, In the News, Strategy Spotlight, etc.)
- Lets editors drag and drop stories between sections to adjust placement
- Warns before replacing a story in a single-slot section (Hero or CEO)
- Validates the draft in real time — flags missing images, empty required slots, and forced hero promotions
- Locks the newsletter when ready and exports a structured JSON payload for the next step (subject line generation)

## How to use it

### Open the tool
Go to: **https://fionadoherty-ibm.github.io/newsletter-assembly/**

No login or installation needed — it runs entirely in your browser.

### Review the draft
Stories are pre-placed based on editorial scoring rules. Check each section and read through the titles and abstracts. Click **Show more** on any card to read the full abstract.

### Reorder or move stories
- Drag the **≡** handle on any card to reorder within a section
- Drag a card into a different section to move it
- If you drag into the **Hero** or **CEO** slot (which can only hold one story), you'll be asked to confirm before the existing story is displaced

### Check the validation bar
The bar at the bottom of the page shows any issues that need resolving before you can lock:
- 🔴 **Red** — blocking issue (e.g. Hero slot empty, missing required image). Must be resolved before locking.
- 🟡 **Amber** — warning (e.g. forced hero promotion active, image needs manual sourcing). Can still lock.
- 🟢 **Green** — all checks passed, ready to lock.

### Skipped stories
Stories that were not approved for this issue appear as pills at the bottom of the page. Hover over a pill to see why it was excluded.

### Lock the newsletter
When you're happy with the layout and all red issues are resolved, click **Lock Newsletter**. This:
- Freezes the layout (drag and drop disabled)
- Exports the final JSON at the bottom of the page — this is the input for the subject line generator
- Shows a green confirmation message

Click **Unlock to edit** if you need to make further changes after locking.

## Updating the content

The stories and section data are defined in the `DRAFT` object at the top of the `newsletter-assembly.html` file. Each new issue requires this data to be updated before the tool is used.

## Questions or issues

Raise an issue on this repository or contact the IBMer News editorial team.
