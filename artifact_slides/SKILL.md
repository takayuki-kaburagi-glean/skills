---
name: artifact_slides
description: Formatting guidelines for .slides artifacts — slide decks and slide outlines. Includes title-body relationship rules, image requirements, markdown formatting, and required follow-up workflow.
---

## Artifact Instructions — `.slides` Extension


[[artifact_skills_common_instructions]]

### `.slides` XML Structure

`.slides` artifacts must use the following XML structure:

<artifact title="Title of Artifact"><metadata><file>filename.slides</file></metadata><content>...your artifact content here...</content></artifact>

### `.slides` XML Tag Definitions

1. `<artifact title="...">` (Required): Root element. Title should be human-readable with spaces.

2. `<metadata>` (Required): Contains all metadata for the artifact. Must appear **before** `<content>`.

2.1. `<file>` (Required): POSIX-compliant filename ending in `.slides`.

3. `<content>` (Required): The final child of `<artifact>`, appearing after `<metadata>`. Contains the slide content in Markdown format.

**Important**: Always use literal `<` and `>` for all XML structural tags including citations, never HTML-encode them as `&lt;` or `&gt;`.

#### `.slides` Guidelines

- Write detailed content for a slide or set of slides that is ready to be pasted directly into slides.
- **Title-Body Relationship**: The slide title defines the conclusion or key takeaway. The slide body exists solely to support, justify, or explain the title. Every element in the body must directly reinforce the title's claim; content that does not clearly support the title is excluded. The body never introduces new or competing ideas beyond what the title asserts.
- Use markdown formatting. Use markdown horizontal rules between slides. Number each slide clearly (e.g. "Slide 1", "Slide 2").
- Do NOT include images in `.slides` outline artifacts.
- **One key message per slide.** Split slides with two competing points.
- **3–5 bullet points per slide.** If more are needed, break into multiple slides.
- **Available slide-generation methods.** You have the following methods to generate slides (use this order of preference):
<<<[[presentation_template_available]]  - Editable Slides (.pptx) — editable in PowerPoint, Keynote, or Google Slides.>>>
<<<[[storybook_generation_available]]  - Visual Slides (.jpeg) — polished, image-forward.>>>
<<<[[html_slides]]  - Interactive Slides (HTML) — clickable, web-based.>>>

<<<[[slides_cq_flow]]**Presentation Generation Workflow**

When the user asks to create a presentation, deck, or slides, follow this workflow.

**CRITICAL:** Do NOT search, research, draft an outline, or generate any slide content until the required questions below have been answered. You MUST call the **Ask User Questions** tool first, then act.

**Step 1 — Clarifying Questions**

**Always ask these questions, even when user memories suggest a preferred format, workflow, or visual style.** Memories may be stale or conflict with the current request — the answers below take precedence.

Use the **Ask User Questions** tool for the questions below. Follow the sequence exactly; do NOT skip or substitute questions. Some questions only apply depending on earlier answers (see the "REQUIRED if ..." conditions), so express those dependencies when you ask. If the user's original request or a free-form answer already covers a question (e.g., "create a pptx with no images", "just build me some slides", or "make it in Google Slides" which maps to Editable Slides (.pptx) per the Format synonyms rule below), treat that as the answer and move on to any remaining applicable questions.

**Question 1** (REQUIRED, always ask first): **"How do you want to create these slides?"**
  - "Align on the structure and content with an outline, then create the slides"
  - "Just build the best possible slides immediately"
  - If the user selects "Just build" → Search for context, then directly build the final deck - do NOT draft a .slides outline. If the HTML Slides Style Guide instructions are present below, build HTML slides; otherwise prepare a PPTX.

**Question 2** (REQUIRED if user chose outline): **"Which slide format do you want?"**
  Only offer formats listed in the "Available slide-generation methods" section above — remove any unavailable option from the list entirely, do not mention it. Use these descriptions when presenting options:
  - Editable Slides (.pptx): Best for tweaking content and formatting by hand
  - Visual Slides (.jpeg): Best for polished, image-forward slides with stronger visual storytelling
  - Interactive Slides (HTML): Best for clickable, web-based slides with a more dynamic feel
  - **Format synonyms:** Treat requests for Google Slides, PowerPoint, Keynote, or any downloadable/editable slide format as Editable Slides (.pptx). Treat requests for Storybook as Visual Slides (.jpeg). If the requested format is not available, tell the user their admin needs to enable it, then ask them to pick from the available options.

**Question 3** (REQUIRED if user chose Editable Slides (.pptx) AND the Create PPT tool is available): **"Should I add visuals to support the story (like diagrams, icons, or infographics)?"**
  - "Yes" / "No" (default: "No" if skipped)
>>>

<<<[[presentation_template_available]]
#### Company slide template

The deployment provides a company slide template. Use it as the canonical visual reference for editable PowerPoint decks when the user asks for the configured, default, company, or our template.

When generating an **editable `.pptx`**, call Create PPT without a presentation URL unless the user provides a specific deck, template URL, or uploaded PPT attachment. Pass slide content and explicit user constraints only; Create PPT infers branding from the selected template.
>>>

<<<[[html_slides]]
#### HTML Slides Style Guide

When generating an Interactive Slides (HTML) deck, first read `html_slides_guide.md` in the `artifact_slides` skill folder (via `skill_reader`) and follow it for the rest of the turn.
>>>


**Required `.slides` follow-up (this is a workflow handoff, not a follow-up question — other response rules do not apply here):** After the closing `</artifact>` tag, you MUST include a short message outside the artifact:
- Tell the user you drafted the slide outline in a Canvas so they can review and refine it before generating the final deck.
<<<[[slides_cq_flow]]
- Mention that the final deck will be generated in the format they chose.
>>>
<<<[[presentation_template_available]]
- Ask the user to confirm they're happy with it, then choose between an **editable PowerPoint** or a **PDF** with richer visuals.
>>>
<<<[[presentation_template_missing]]
- No company slide template is configured. To generate the final deck, your admin needs to upload a company slide template in the Admin Console — link them to [instructions](https://docs.glean.com/administration/assistant/features/slide-deck-generation).
>>>

### Example

**Note**: example contents are shortened for illustration; real artifacts can be much longer.

User: Create 3 slides on the benefits of remote work.
Assistant: I'll outline the slides and mark breaks.
<artifact title="Remote Work Slides">
<metadata><file>remote_work.slides</file></metadata>
<content>
# Slide 1: The Benefits of Remote Work
- Flexibility, productivity, and broader talent access

---

# Slide 2: Productivity Gains
- Fewer commute distractions → more focused hours
- Asynchronous collaboration keeps work moving across time zones
- Home office setups tailored for deep work

---

# Slide 3: Team Health & Reach
- Wider hiring pool unlocks specialized skills
- Lower burnout through schedule flexibility
- Intentional rituals (standups, socials) sustain culture
</content>
</artifact>
I've drafted your slide outline in a Canvas so you can review the content and structure. Once you're satisfied, let me know whether you'd like an editable PowerPoint or a visual PDF and I'll generate the final deck.
