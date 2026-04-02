# Design Evaluation Criteria

## Scoring System

4 criteria, each scored 1–10. Scores must be integers.

**Weighted Average** = (Design Quality × 2 + Originality × 2 + Craft × 1 + Functionality × 1) / 6

Design Quality and Originality carry double weight because Claude naturally excels at technical execution (Craft, Functionality) but struggles with cohesive vision and avoiding template-like outputs.

---

## Criterion 1: Design Quality (Weight: 2×)

**Core question**: Does the design feel like a coherent whole rather than a collection of parts?

### What to evaluate
- Do colors, typography, layout, and imagery combine to create a distinct mood or identity?
- Is there a unified visual language across all sections?
- Does the design have a clear emotional direction (minimal, bold, warm, corporate, playful, etc.)?
- Do elements feel like they belong together — or like they were assembled from different templates?

### Score anchors
| Score | Description |
|-------|-------------|
| 1–3 | Disconnected elements, no visual theme, looks like random components thrown together |
| 4–5 | Some visual consistency but no distinct identity — could be any generic website |
| 6–7 | Clear visual direction with minor inconsistencies between sections |
| 8–9 | Strong cohesive identity — every element reinforces the same mood and story |
| 10 | Museum-quality coherence — nothing could be added or removed without breaking the whole |

### Anti-patterns (penalize heavily)
- Sections that look like they belong to different websites
- Inconsistent visual treatment (some sections with shadows, others flat, no clear reason)
- No clear visual hierarchy across the full page
- Color palette that doesn't tell a story (random accent colors)

---

## Criterion 2: Originality (Weight: 2×)

**Core question**: Does this design make custom, intentional decisions — or does it rely on defaults and templates?

### What to evaluate
- Are there distinctive typographic choices (not system fonts)?
- Does the layout break away from predictable patterns?
- Are color choices intentional and memorable?
- Would you remember this design after seeing 20 others?

### Score anchors
| Score | Description |
|-------|-------------|
| 1–3 | Pure template — Inter/Roboto, purple gradient hero, white card grid, stock imagery |
| 4–5 | Some custom decisions but overwhelmingly safe and forgettable |
| 6–7 | Notable creative choices in 1–2 areas (typography OR layout OR color) but not throughout |
| 8–9 | Distinctive across multiple dimensions — you'd recognize this design in a lineup |
| 10 | Genuinely novel approach that sets a new direction |

### AI slop patterns (penalize HEAVILY — subtract 2–3 points if present)
- Purple/blue gradients over white cards
- Inter, Roboto, Arial, or system font stack as primary typeface
- Generic rounded-corner card grid as main content pattern
- Cookie-cutter hero section (big heading + subtitle + CTA button + stock illustration)
- Evenly distributed pastel palette with no dominant color
- Glassmorphism or neumorphism used as decoration without purpose
- "Trusted by" logo bar as design filler

### What "good" looks like
- Typography that has personality (display font paired with refined body font)
- Unexpected layout decisions (asymmetry, overlapping elements, creative grid breaks)
- Bold color choices with clear hierarchy (dominant + accent, not 5 equal colors)
- Design elements that reflect the product's actual purpose or brand

---

## Criterion 3: Craft (Weight: 1×)

**Core question**: Is the technical execution precise and consistent?

### What to evaluate
- **Typography hierarchy**: Clear distinction between H1 → H2 → H3 → body → caption. Consistent font sizes and weights.
- **Spacing consistency**: Uniform padding and margins. Sections follow a spacing scale (8px grid or similar). No random gaps.
- **Color harmony**: Intentional palette. Sufficient contrast ratios (WCAG AA: 4.5:1 for text, 3:1 for large text). No clashing colors.
- **Alignment**: Elements align to a grid. Left edges line up. Consistent gutters between columns.
- **Detail work**: Consistent border radii. Proper icon sizing. No orphaned text lines. Image aspect ratios preserved.

### Score anchors
| Score | Description |
|-------|-------------|
| 1–3 | Broken layout, overlapping elements, illegible text, missing alignment |
| 4–5 | Functional but sloppy — inconsistent spacing, mixed radii, misaligned elements |
| 6–7 | Generally clean with minor issues (one section with odd spacing, slight misalignment) |
| 8–9 | Pixel-perfect execution — every detail is intentional and consistent |
| 10 | Flawless craft that reveals new details on close inspection |

### Common issues to check
- Text clipping or overflow
- Inconsistent padding between similar sections
- Mixed border-radius values without clear system
- Icons at different sizes or weights
- Color contrast failures (light text on light background)
- Orphaned words/lines in headings

---

## Criterion 4: Functionality (Weight: 1×)

**Core question**: Can a user understand and navigate this design without guessing?

### What to evaluate
- **3-second test**: Can a new user understand the page's purpose within 3 seconds?
- **Interactive affordance**: Do buttons look like buttons? Do links look like links? Are clickable areas obvious?
- **Information hierarchy**: What should the user read first, second, third? Is the flow clear?
- **Primary action**: Is there one clear CTA? Or is the user overwhelmed with choices?
- **Navigation clarity**: Can the user orient themselves and find what they need?

### Score anchors
| Score | Description |
|-------|-------------|
| 1–3 | Confusing purpose, no clear CTA, user would leave immediately |
| 4–5 | Purpose is guessable but not instant. Multiple competing CTAs. Unclear flow. |
| 6–7 | Clear purpose and primary action. Minor navigation ambiguity. |
| 8–9 | Intuitive flow — user knows exactly what to do and where to look at every point |
| 10 | Effortless UX — the design anticipates user needs before they arise |

### Anti-patterns
- No clear primary action (everything looks equally important)
- Hidden or ambiguous navigation
- Information overload (too many sections competing for attention)
- Dead ends (sections that don't lead anywhere)
- Decorative elements that look interactive (or vice versa)

---

## Evaluator Behavioral Rules

### Independence
- You are an independent critic, NOT a cheerleader
- You have NOT seen the plan or implementation code — judge only what you see
- Do not assume good intentions — judge the visible result

### Scoring discipline
- Scores MUST spread across the range. If all 4 scores are within 1 point of each other, you are not being honest enough.
- A score of 5/10 is acceptable and means "below average but functional"
- Do not cluster scores around 7 (the "polite" zone)
- A first-round design getting 7+ on Originality should be rare — most first attempts are generic

### Feedback specificity
- Every improvement suggestion must be SPECIFIC and ACTIONABLE
- BAD: "Improve the spacing" / "Make it more original" / "Add visual interest"
- GOOD: "The hero section has 16px padding while the features section has 48px — standardize to 48px using the spacing/lg token"
- GOOD: "Replace the Inter heading with a display serif (e.g., Playfair Display) to create contrast with the geometric body text"
- GOOD: "The 3-column card grid is the most common AI pattern — try a 2-column asymmetric layout with the primary plan taking 60% width"
- Reference specific elements by their position or name when possible

### Round-over-round tracking (Round 2+)
- Explicitly acknowledge what IMPROVED from the previous round
- Explicitly flag what REGRESSED (got worse) from the previous round
- If the Doer ignored a previous suggestion, note it
- Compare scores against previous round and justify any changes

### Top 3 improvements
- Rank by IMPACT — what single change would improve the design the most?
- Focus on the weakest criterion first
- Each improvement should be independently actionable (not "first do X, then Y depends on X")

---

## Output Format

The evaluator MUST return results in this exact format:

```
### Scores
- Design Quality: X/10 — [2-3 sentence justification]
- Originality: X/10 — [2-3 sentence justification]
- Craft: X/10 — [2-3 sentence justification]
- Functionality: X/10 — [2-3 sentence justification]
- **Weighted Average: X.X/10**

### Top 3 Improvements (ranked by impact)
1. [Specific, actionable improvement with element references]
2. [Specific, actionable improvement with element references]
3. [Specific, actionable improvement with element references]

### What Worked (Round 2+ only)
- [Improvement from previous round that landed well]
- [Another improvement that worked]

### What Regressed (Round 2+ only, if applicable)
- [Anything that got worse compared to previous round]

### Detailed Critique
[2-3 paragraph analysis covering the overall design direction, strongest and weakest aspects, and the single most important thing to change next]
```
