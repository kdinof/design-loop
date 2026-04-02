# Evaluator Calibration Examples

These examples anchor the evaluator's judgment across the scoring range. Study them before evaluating — they define what each score level looks and feels like.

---

## Example 1: Generic AI Design (Low — scores 3–5)

### Description
A SaaS landing page with:
- Hero: large heading in Inter Bold, subtitle in Inter Regular, purple-to-blue gradient background, white CTA button, abstract 3D illustration on the right
- Features: 3-column card grid with white cards, light gray background, rounded corners (12px), each card has a purple icon, heading, and 2 lines of body text
- Pricing: 3 equal-width pricing cards, center one highlighted with purple border, "Most Popular" badge
- Footer: simple 4-column link grid, dark background

### Evaluation

#### Scores
- Design Quality: 3/10 — The design has no cohesive identity. The purple gradient hero feels disconnected from the flat white feature cards. The 3D illustration style doesn't match anything else on the page. It looks assembled from different Figma Community templates.
- Originality: 2/10 — This is the default AI output: Inter font, purple gradient, white card grid, generic 3D illustration. Every element is the most common choice for its category. You could generate this by typing "SaaS landing page" into any AI tool.
- Craft: 5/10 — Spacing is consistent within sections (16px card padding, 24px between cards). Typography hierarchy exists (48px/24px/16px). But the heading-to-body contrast is weak (Inter Bold vs Inter Regular looks nearly identical at distance). Card grid has equal gutters. Technically functional.
- Functionality: 6/10 — Purpose is clear within 3 seconds. CTA is visible. Pricing comparison works. But there's no clear information flow between sections — features and pricing feel like independent blocks rather than a narrative.
- **Weighted Average: 3.5/10**

#### Top 3 Improvements
1. Replace Inter with a distinctive display font for headings (e.g., Cabinet Grotesk, General Sans, or a serif like Fraunces) — this single change would immediately lift Originality by 2+ points
2. Kill the purple gradient and 3D illustration. Choose a real color direction: warm earth tones, monochrome with one accent, or dark mode with neon. Commit to it across ALL sections, not just the hero.
3. Break the 3-column card grid. Try a 2-column layout with one featured card at 60% width, or a horizontal scrolling row, or a comparison table instead of cards.

---

## Example 2: Competent But Unremarkable (Mid — scores 5–7)

### Description
A project management tool landing page with:
- Hero: dark navy (#1a1a2e) background, white heading in DM Sans Bold (56px), warm orange (#f97316) CTA button, product screenshot in a browser mockup with subtle drop shadow
- Features: alternating left-right layout (image + text, text + image) on white background, each feature with an orange icon accent, DM Sans for all text
- Social proof: single row of grayscale company logos on light gray bar
- Pricing: 2 plans side-by-side with clear visual hierarchy (free plan muted, paid plan with orange border and filled CTA)
- Footer: 3-column layout on dark navy, matching the hero

### Evaluation

#### Scores
- Design Quality: 6/10 — The dark navy + orange palette creates a consistent mood across hero and footer, and the alternating feature layout provides rhythm. However, the white middle sections break the dark theme's momentum — the page has two identities (dark hero/footer vs white body). The social proof bar feels like filler.
- Originality: 5/10 — DM Sans is a step above Inter but still common. The alternating feature layout is a well-worn pattern. The dark hero + white body is the most standard SaaS approach. Orange is a valid accent but the execution is safe. The browser mockup is a cliché. Nothing here would make you stop scrolling.
- Craft: 7/10 — Good typography scale (56/32/20/16). Consistent 48px section padding. The product screenshot has proper shadow and border radius. Orange is used consistently for CTAs and accents only. One issue: the alternating sections have inconsistent image sizes (first image is taller than the second), breaking the visual rhythm.
- Functionality: 7/10 — Clear purpose immediately. Hero CTA is prominent. Features explain the product well. Pricing is easy to compare with clear recommended option. The only issue: too much scrolling before reaching pricing — the social proof bar adds length without much value at this position.
- **Weighted Average: 6.0/10**

#### Top 3 Improvements
1. Commit to the dark theme throughout — extend the navy background to feature sections with white text, using orange sparingly for highlights. This would unify the page identity and immediately boost Design Quality.
2. Replace the browser mockup with an actual UI screenshot at higher fidelity, or use a custom illustration that matches the geometric style of the icons. The mockup is generic filler.
3. Replace the alternating left-right feature layout with something less predictable: a bento grid, overlapping cards, or a single scrolling feature showcase with transitions between features.

---

## Example 3: Distinctive and Polished (High — scores 8–9)

### Description
A creative agency portfolio page with:
- Hero: full-bleed black background, oversized serif heading in Editorial New (120px, cream #faf4e8 color) with one word in italic, no subtitle — just the heading and a small circular "scroll" indicator with animated arrow
- Work showcase: asymmetric grid — large featured project (65% width) next to two stacked smaller projects (35% width), each with a hover-revealed project title in monospace type (JetBrains Mono). Generous 32px gaps. Images have slight desaturation with color returning on hover.
- About: split layout — left half is a vertical text block with pull quote in oversized italic serif (48px), right half is a full-height team photo with grain overlay. Cream background throughout.
- Contact: minimal — centered email address in 64px monospace, period at the end as a design element. Black background returns, creating bookend symmetry with hero.

### Evaluation

#### Scores
- Design Quality: 9/10 — Exceptional coherence. The black ↔ cream palette creates a clear rhythm (dark-light-dark). The serif + monospace type pairing is consistent and intentional throughout. The desaturated images create a unified visual treatment. Every section feels like a chapter of the same story. The bookend structure (black hero → cream body → black contact) is a deliberate narrative device.
- Originality: 8/10 — Strong distinctive choices: oversized serif typography at 120px is bold and memorable. The monospace project titles create unexpected contrast. The desaturation-to-color hover effect is creative. The contact section (just an email in 64px monospace) is a statement. One point deducted because the black-and-cream editorial aesthetic, while well-executed, has been trending in creative agency sites — it's distinctive but not novel.
- Craft: 9/10 — The asymmetric grid is precisely proportioned (65/35 split with consistent 32px gaps). Typography scale is deliberate: 120px hero → 48px pull quote → 24px body → 14px monospace labels. The grain overlay on the team photo adds texture without affecting readability. Image desaturation is uniform. Only minor issue: the "scroll" indicator could be positioned more intentionally relative to the heading's baseline.
- Functionality: 7/10 — The portfolio structure is clear and the work showcase hierarchy works well (featured + secondary). However, there's no visible navigation — a first-time visitor has no way to jump to the work section or about section without scrolling. The contact section relies on the user knowing to click an email address. For a creative agency, this minimal approach may be intentional, but it sacrifices some usability.
- **Weighted Average: 8.3/10**

#### Top 3 Improvements
1. Add a subtle fixed navigation — even just 3-4 small text links in the top right (Work / About / Contact) in monospace to match the design language. This preserves the minimal aesthetic while solving the navigation gap. Would push Functionality to 8+.
2. The "scroll" indicator in the hero needs refinement — align it to the right margin of the heading text, and consider making it a text element ("scroll" in monospace) rather than a generic circle+arrow. Keep the design language consistent.
3. Consider adding one more color to the palette — a single muted accent (terracotta, olive, or dusty blue) used only for interactive states (hover, active links). The current black+cream is coherent but adding a purposeful third color would elevate the sophistication.

---

## How to Use These Examples

1. **Before scoring**: Re-read the example closest to what you're evaluating. Use it as a mental benchmark.
2. **Score calibration**: If the design you're evaluating is clearly better than Example 1 but not as strong as Example 2, your scores should fall between those ranges (4–6).
3. **Feedback calibration**: Match the specificity level of these examples. Every critique names specific elements, measurements, or design tokens.
4. **Avoid drift**: In later rounds, compare not just against the previous round but also against these anchors. A Round 3 design at 6/10 on Originality should still match the "notable creative choices in 1–2 areas" description from the rubric.
