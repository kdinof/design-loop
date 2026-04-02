---
name: design-loop
description: "Iterative Figma design creation using the Generator-Evaluator harness pattern. Runs a Planner → Doer → Evaluator loop for N rounds, cloning the frame each round so the user sees design evolution side by side. Use when the user wants to iteratively refine a Figma design, says 'design loop', 'iterate on design', 'refine this layout', 'run design rounds', or wants AI-driven design iteration with evaluation feedback."
disable-model-invocation: false
---

# Design Loop — Iterative Figma Design Harness

A 3-role loop (Planner → Doer → Evaluator) that iteratively creates and refines Figma designs across N rounds. Each round clones the previous frame so the user sees the full evolution side by side.

**Communicate with the user in Russian. All internal artifacts (plans, evaluations, code) in English.**

## Progress Reporting

At every phase transition, show the user a progress tracker so they always know where they are. Use this exact format:

```
───────────────────────────────────────
  Design Loop — Раунд {N}/{maxRounds}
───────────────────────────────────────
  ✅ Инициализация
  ✅ Планирование          ← completed phases
  ▶️ Реализация в Figma     ← current phase
  ○ Аннотация
  ○ Оценка
  ○ Итог раунда
───────────────────────────────────────
```

Rules:
- `✅` = completed phase
- `▶️` = current phase (always exactly one)
- `○` = upcoming phase
- Show the tracker **before** starting each new phase
- During **Initialization** (before rounds begin), use this header instead: `Design Loop — Подготовка`
- During **Final Summary**, use: `Design Loop — Финальный итог`
- Keep the tracker compact — it should be a quick glance, not a wall of text. Add one short status line below the tracker if needed (e.g., "Создаю фрейм для раунда 2..." or "Оценщик анализирует дизайн...")

The full phase sequence per round:
1. **Планирование** (Planner — Phase A)
2. **Версионирование фрейма** (Frame Versioning — Phase B)
3. **Реализация в Figma** (Doer — Phase C)
4. **Аннотация** (Annotation — Phase C½)
5. **Оценка** (Evaluator — Phase D)
6. **Итог раунда** (Loop Control — Phase E)

For multi-round runs, phases from completed rounds should NOT be shown — only show phases for the current round plus a summary line for completed rounds. Example for Round 2:

```
───────────────────────────────────────
  Design Loop — Раунд 2/3
───────────────────────────────────────
  ✅ Раунд 1 — 5.2/10
  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
  ✅ Планирование
  ✅ Версионирование фрейма
  ▶️ Реализация в Figma
  ○ Аннотация
  ○ Оценка
  ○ Итог раунда
───────────────────────────────────────
```

## 1. Prerequisites

Before starting the loop, load these skills:

1. **MANDATORY**: Load [figma-use](../figma-use/SKILL.md) — required before every `use_figma` call
2. **RECOMMENDED**: Load [figma-generate-design](../figma-generate-design/SKILL.md) — for full-screen builds in Round 1
3. **OPTIONAL**: Load [frontend-design](../../plugins/cache/claude-plugins-official/frontend-design/SKILL.md) — for distinctive aesthetic direction (avoids AI slop)

## 2. Arguments

Parse from user input:

- **brief** (required): The design task description. Example: "Create a SaaS pricing page with 3 tiers"
- **--rounds N** (optional, default: 3): Number of iteration rounds (1–5)
- **--criteria-override** (optional): Path to custom evaluation criteria file

If the user provides no brief, ask for one before proceeding.

## 3. Initialization

Execute these steps before entering the loop:

### 3a. Design System Setup

On **first ever launch** (no DS analysis saved in memory), ask the user for their design system:

> Привет! Для качественной работы мне нужна ссылка на вашу дизайн-систему в Figma (файл с компонентами, переменными и стилями). Пожалуйста, вставьте URL.

Once the user provides the URL, extract the **file key** from it and perform a full DS analysis:

#### DS Analysis Workflow

1. **Get file structure** — use `figma_get_file_data` or `use_figma` to list all pages in the DS file
2. **Catalog components** — use `figma_search_components` or `figma_get_library_components` to extract:
   - Component names and keys
   - Component set variants (e.g., Button: Size=S/M/L, Style=Primary/Secondary)
   - Component properties (text overrides, boolean toggles, instance swaps)
3. **Catalog variables** — use `figma_get_variables` to extract:
   - Variable collections (e.g., "Colors", "Spacing", "Typography")
   - Variable names, types, and values per mode (e.g., light/dark)
   - Aliases (variables referencing other variables)
4. **Catalog styles** — use `figma_get_styles` and `figma_get_text_styles` to extract:
   - Color styles (name → hex)
   - Text styles (name → font family, size, weight, line height)
   - Effect styles (shadows, blurs)
5. **Build a structured DS summary** covering:
   - Color palette: primary, secondary, neutral, semantic colors with exact values
   - Typography scale: heading/body/caption styles with font families, sizes, weights
   - Spacing scale: token names and pixel values
   - Component inventory: grouped by category (buttons, inputs, cards, navigation, etc.) with available variants
   - Layout patterns: common containers, grids, section structures found in the DS

#### Save DS analysis to memory

Save the full analysis to a memory file `reference_design_system_analysis.md` with type `reference`. This includes:
- The Figma URL and file key
- The complete structured summary from step 5
- A timestamp of when the analysis was performed

On **subsequent launches**, check if the DS analysis memory file exists. If it does, read it and skip the setup. If the user provides a new DS link, re-run the analysis and overwrite the saved memory.

### 3b. Load evaluation criteria

1. Check if the project has a custom criteria file: `.claude/design-loop-criteria.md`
2. If not found, load the default: [references/evaluation-criteria.md](references/evaluation-criteria.md)
3. Also load: [references/calibration-examples.md](references/calibration-examples.md)

### 3c. Discover Figma file context

Run a read-only `use_figma` call to discover the **target design file** state (the file the user is designing in — NOT the DS file):

```js
const pages = figma.root.children.map(p => ({ id: p.id, name: p.name }));
const currentPage = figma.currentPage;
const existingFrames = currentPage.children
  .filter(n => n.type === 'FRAME' || n.type === 'SECTION')
  .map(n => ({ id: n.id, name: n.name, x: n.x, y: n.y, width: n.width, height: n.height }));
return { pages, currentPageId: currentPage.id, currentPageName: currentPage.name, existingFrames };
```

### 3d. Initialize loop state

Maintain this state throughout the session (do NOT write to disk — keep in context):

```
## Loop State
- round: 1
- maxRounds: [parsed from args]
- brief: "[user's design brief]"
- figmaFileKey: "[extracted from Figma file URL or discovered via figma_list_open_files]"
- dsFileKey: "[from DS analysis]"
- frameIds: []
- scores: []
- evaluatorFeedback: []
- designSystemContext: { componentKeys: {}, variableKeys: {}, styleKeys: {}, dsAnalysis: "[loaded from memory]" }
```

### 3d. Inform the user

Show the initialization progress tracker and tell the user (in Russian):
- What will be created
- How many rounds
- That each round creates a separate frame for comparison

```
───────────────────────────────────────
  Design Loop — Подготовка
───────────────────────────────────────
  ✅ Загрузка критериев оценки
  ✅ Обнаружение контекста Figma
  ✅ Инициализация состояния
───────────────────────────────────────
```

Then immediately show the Round 1 tracker with Планирование as the current phase.

---

## 4. Round Execution

Repeat this section for each round from 1 to maxRounds.

---

### Phase A: PLANNER

**Show progress tracker** with "Планирование" as `▶️` current phase.

The Planner produces an actionable design plan. It runs inline (in the main session).

#### Round 1 — Initial plan

Inputs:
- User's design brief
- Design system context (components, variables, styles discovered in Step 3b)
- Aesthetic direction from `frontend-design` skill (if loaded)

Output a plan covering:
1. **Page structure**: List all sections top-to-bottom (e.g., Hero → Features → Pricing → FAQ → Footer)
2. **Component mapping**: Which design system components to use for each section (by name/key)
3. **Visual direction**: Color palette, typography choices, layout approach, mood
4. **Variable usage**: Which design tokens (spacing, colors, radii) to bind
5. **Expected dimensions**: Approximate frame width and section heights

**Critical rule**: The plan MUST be SPECIFIC and ACTIONABLE. Never write vague instructions.
- BAD: "Create an attractive hero section"
- GOOD: "Create a hero section: 1440×720px, dark background using color/bg/primary variable, heading in Editorial New 72px cream (#faf4e8), subtitle in DM Sans 20px gray-300, orange CTA button using the Button component (key: abc123) with variant Style=Primary"

#### Round N>1 — Improvement plan

Inputs:
- User's original brief
- Evaluator feedback from Round N-1 (scores + top 3 improvements + detailed critique)
- Previous round's scores
- Current design system context

Output a plan covering:
1. **What to change**: Specific modifications addressing each of the evaluator's Top 3 improvements
2. **What to keep**: Elements that scored well and should NOT be changed
3. **Risk assessment**: Which changes might cause regressions in other criteria
4. **Priority order**: Execute highest-impact changes first

**Critical rule**: Address ALL of the evaluator's Top 3 improvements. If you disagree with a suggestion, explain why and propose an alternative — do not silently ignore feedback.

---

### Phase B: FRAME VERSIONING

**Show progress tracker** with "Версионирование фрейма" as `▶️` current phase.

Before the Doer executes, create or clone the target frame.

#### Round 1 — Create initial frame

Use `use_figma` to create a new frame positioned away from existing content:

```js
// Find clear position to the right of all existing content
const allNodes = figma.currentPage.children;
let maxRight = 0;
for (const node of allNodes) {
  const right = node.x + node.width;
  if (right > maxRight) maxRight = right;
}

const frame = figma.createFrame();
frame.name = "Round 1";
frame.x = allNodes.length > 0 ? maxRight + 200 : 0;
frame.y = 0;
frame.resize(1440, 900); // Default desktop width, will grow with content
frame.fills = [{ type: 'SOLID', color: { r: 1, g: 1, b: 1 } }];

// Set up auto-layout for vertical stacking
frame.layoutMode = 'VERTICAL';
frame.primaryAxisSizingMode = 'AUTO'; // Height hugs content
frame.counterAxisSizingMode = 'FIXED'; // Width stays at 1440
frame.itemSpacing = 0;

return { frameId: frame.id, name: frame.name, x: frame.x, y: frame.y };
```

Store `frameId` in loop state under `frameIds.round1`.

#### Round N>1 — Clone previous frame

Use `use_figma` to clone the previous round's frame:

```js
const prevFrame = await figma.getNodeByIdAsync("PREVIOUS_FRAME_ID");
if (!prevFrame) throw new Error("Previous frame not found: PREVIOUS_FRAME_ID");

const clone = prevFrame.clone();
clone.name = "Round N";
clone.x = prevFrame.x + prevFrame.width + 200;
// y stays the same — all rounds align horizontally

return { clonedFrameId: clone.id, name: clone.name, x: clone.x, y: clone.y, width: clone.width, height: clone.height };
```

Store the new `clonedFrameId` in loop state under `frameIds.roundN`.

---

### Phase C: DOER

**Show progress tracker** with "Реализация в Figma" as `▶️` current phase.

The Doer executes the Planner's instructions in Figma. It runs inline (needs MCP tool access).

#### Round 1 — Build from scratch

Follow the `figma-generate-design` workflow:

1. **Work section by section**. One major section per `use_figma` call (Hero, Features, Pricing, etc.)
2. **Use design system components** where available. Import by key, instantiate, set properties.
3. **Bind variables** instead of hardcoding values. Import variables by key, bind to fills/spacing/radii.
4. **Return all node IDs** from every `use_figma` call.
5. **Validate after each section**:
   - `get_metadata` on the frame → check structure, node counts, hierarchy
   - `get_screenshot` of the section → check visual correctness (no clipping, no overlap)
6. **Fix issues** before moving to the next section. Max 3 fix iterations per section.

#### Round N>1 — Targeted modifications

The cloned frame already has the full design. Make targeted changes per the Planner's improvement plan:

1. **Locate target nodes** in the cloned frame using `use_figma` with `findOne`/`findAll`.
2. **Make one change per `use_figma` call**:
   - Swap component variants
   - Rebind variables (colors, spacing)
   - Adjust typography (font family, size, weight — remember to load fonts first)
   - Rearrange or resize sections
   - Add/remove elements
3. **Validate after each significant change** with `get_metadata` and/or `get_screenshot`.

**Critical rule**: Do NOT rebuild the entire frame from scratch. The clone already has the previous round's work. Make surgical edits only.

---

### Phase C½: ANNOTATION

**Show progress tracker** with "Аннотация" as `▶️` current phase.

After the Doer completes each round, create a text annotation block directly below the round's frame. This gives the user a persistent, in-Figma changelog explaining what was changed and why.

#### What the annotation contains

1. **Title**: "Round N — [short theme]" (e.g., "Round 1 — Типографика и контраст")
2. **What changed** (bulleted list): Each specific modification made by the Doer in this round, with concrete values (font sizes, colors, spacing, etc.)
3. **Why** (paragraph): The reasoning — for Round 1 this comes from the Planner's analysis of the original design; for Round N>1 this references the Evaluator's feedback from the previous round

#### How to create the annotation

Use `use_figma` to create a styled frame below the round's frame:

```js
await figma.loadFontAsync({ family: "Inter", style: "Regular" });
await figma.loadFontAsync({ family: "Inter", style: "Bold" });

const roundFrame = await figma.getNodeByIdAsync("ROUND_FRAME_ID");

// Container
const container = figma.createFrame();
container.name = "Round N — Annotation";
container.x = roundFrame.x;
container.y = roundFrame.y + roundFrame.height + 32;
container.resize(roundFrame.width, 10); // height will auto-grow
container.layoutMode = "VERTICAL";
container.primaryAxisSizingMode = "AUTO";
container.counterAxisSizingMode = "FIXED";
container.itemSpacing = 8;
container.paddingTop = 16;
container.paddingBottom = 16;
container.paddingLeft = 16;
container.paddingRight = 16;
container.fills = [{ type: 'SOLID', color: { r: 0.96, g: 0.96, b: 0.97 } }];
container.cornerRadius = 12;

// Title (Bold)
const titleNode = figma.createText();
titleNode.fontName = { family: "Inter", style: "Bold" };
titleNode.characters = "ANNOTATION_TITLE";
titleNode.fontSize = 14;
titleNode.lineHeight = { unit: "PIXELS", value: 20 };
titleNode.fills = [{ type: 'SOLID', color: { r: 0.1, g: 0.1, b: 0.12 } }];
titleNode.textAutoResize = "HEIGHT";
container.appendChild(titleNode);
titleNode.layoutSizingHorizontal = "FILL";

// Body (Regular)
const bodyNode = figma.createText();
bodyNode.fontName = { family: "Inter", style: "Regular" };
bodyNode.characters = "ANNOTATION_BODY";
bodyNode.fontSize = 12;
bodyNode.lineHeight = { unit: "PIXELS", value: 18 };
bodyNode.fills = [{ type: 'SOLID', color: { r: 0.3, g: 0.3, b: 0.35 } }];
bodyNode.textAutoResize = "HEIGHT";
container.appendChild(bodyNode);
bodyNode.layoutSizingHorizontal = "FILL";

return { createdNodeIds: [container.id, titleNode.id, bodyNode.id] };
```

#### Annotation content guidelines

- **Language**: Russian (user-facing artifact)
- **"What changed" section**: Use bullet points (•). Include specific values: pixel sizes, hex colors, spacing amounts. Reference the element by its visible role (e.g., "заголовок", "кнопка CTA", "подзаголовок"), not by node ID or technical name.
- **"Why" section**: For Round 1 — explain what problems in the original design motivated the changes. For Round N>1 — directly reference the Evaluator's feedback (e.g., "Оценщик указал на слабый контраст кнопки на синем фоне — белый цвет решает эту проблему").
- **Keep it concise**: Max ~150 words per annotation. The goal is a quick-scan changelog, not an essay.

---

### Phase D: EVALUATOR (Sub-Agent)

**Show progress tracker** with "Оценка" as `▶️` current phase. Add status line: "Оценщик анализирует дизайн..."

The Evaluator MUST run as a **separate sub-agent** to prevent self-evaluation bias. The key insight from the Anthropic harness article: models confidently praise their own work even when objectively mediocre. Separating the evaluator into an independent context window is the only reliable fix.

#### What the Evaluator receives

**YES — pass these to the Evaluator**:
- The Figma file key (from the file URL)
- The frame node ID for the current round (from loop state)
- The user's original design brief
- The evaluation criteria and calibration examples
- Previous round scores and feedback (if Round 2+)
- **The DS analysis summary** (color palette, typography scale, spacing tokens, component inventory) — the Evaluator needs this to assess DS conformance

**NO — do NOT pass these to the Evaluator**:
- The Planner's plan or reasoning
- The Doer's `use_figma` code or implementation details
- Specific component keys or variable IDs (the Evaluator gets the DS *visual spec*, not implementation handles)
- Any `get_metadata` output (node structure reveals implementation)

#### Evaluator's visual inspection workflow

The Evaluator takes its own screenshots directly from Figma using `get_screenshot`. This gives it independence — it decides what to inspect, not the Doer. The Evaluator should:

1. **Full-frame screenshot** — overall impression, coherence, visual hierarchy
2. **Section-level screenshots** — zoom into individual sections (hero, features, pricing, footer) to check details: text readability, spacing precision, icon sizing, contrast
3. **Problem areas** — if something looks off in the full screenshot, zoom in to confirm

The Evaluator must ONLY use `get_screenshot` for visual inspection. It must NOT use `get_metadata` or `use_figma` — these would reveal implementation details and create evaluation bias.

#### Spawning the Evaluator

Use the Agent tool to spawn a sub-agent with this prompt:

```
You are an independent design evaluator. You have NOT seen the implementation plan or code — you are judging purely on the visual result.

## Your Tools
You have access to `get_screenshot` to inspect the Figma design. Use it to:
1. Take a full-frame screenshot of the design (node ID: {FRAME_NODE_ID}, file key: {FILE_KEY})
2. Take section-level screenshots to inspect details — zoom into individual areas
3. Investigate anything that looks off at full scale

IMPORTANT: Do NOT use `get_metadata` or `use_figma`. You are evaluating the visual result only, not the implementation. Seeing node structure or variable names would bias your judgment.

## Design Brief
{USER_BRIEF}

## Evaluation Round
Round {N} of {MAX_ROUNDS}

## Previous Round Scores
{PREVIOUS_SCORES or "N/A — this is Round 1"}

## Design System Reference
The design was built using the following design system. Use this as the source of truth for evaluating visual consistency and DS conformance:

{DS_ANALYSIS_SUMMARY — include: color palette with exact values, typography scale with font families/sizes/weights, spacing tokens, component inventory with variants}

When evaluating, check:
- Do the colors in the design match the DS palette? Flag any colors that appear custom/off-palette.
- Does the typography follow the DS text styles? Check font families, sizes, weights, line heights.
- Is the spacing consistent with DS tokens? Look for values that don't align with the spacing scale.
- Are DS components used correctly? Check that buttons, inputs, cards match their DS variants visually.
- Is the overall visual language consistent with the DS identity?

## Evaluation Criteria
{CONTENTS OF evaluation-criteria.md}

## Calibration Examples
{CONTENTS OF calibration-examples.md}

## Your Task
1. Take screenshots of the design (full frame + individual sections)
2. Evaluate against the criteria above
3. Evaluate DS conformance — flag deviations from the design system
4. Follow the Output Format specified in the evaluation criteria EXACTLY
5. Add a "### DS Conformance" section after the Detailed Critique:
   - List specific DS violations (wrong colors, non-DS fonts, off-scale spacing, etc.)
   - Note where DS components appear correctly used
   - Score DS conformance qualitatively: "Excellent" / "Good" / "Needs Work" / "Poor"

Be HARSH — a 5/10 is acceptable. Do not cluster scores around 7.
Every improvement must be SPECIFIC with element references.
{IF ROUND > 1: "Compare against previous round scores. Acknowledge what improved and what regressed."}
```

#### Processing Evaluator output

Parse the evaluator's response to extract:
1. **Scores**: Design Quality, Originality, Craft, Functionality, Weighted Average
2. **Top 3 Improvements**: Store for the next round's Planner
3. **What Worked / What Regressed**: Store for tracking
4. **Detailed Critique**: Store for context
5. **DS Conformance**: Store DS violations and conformance rating — feed these to the Planner so the next round can fix DS deviations

Update loop state with scores and feedback.

**Show the evaluation results to the user** (in Russian) — they should see the scores, key feedback, and DS conformance summary after each round.

---

### Phase E: LOOP CONTROL

**Show progress tracker** with "Итог раунда" as `▶️` current phase.

After the Evaluator completes Round N:

1. **If N < maxRounds AND weighted average < 9.0**:
   - Show a round summary with scores and the Top 3 improvements briefly
   - Inform the user: "Раунд N завершён. Оценка: X.X/10. Переходим к раунду N+1."
   - Proceed to Phase A (Planner) for Round N+1, passing evaluator feedback

2. **If N == maxRounds OR weighted average >= 9.0**:
   - Proceed to Final Summary (Section 5)
   - If early termination (avg >= 9.0): tell the user the design reached excellent quality

---

## 5. Final Summary

**Show final progress tracker:**

```
───────────────────────────────────────
  Design Loop — Финальный итог
───────────────────────────────────────
  ✅ Раунд 1 — X.X/10
  ✅ Раунд 2 — X.X/10
  ✅ Раунд 3 — X.X/10
  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
  Итог: X.X → X.X (+X.X)
───────────────────────────────────────
```

After all rounds complete, present the user with:

### Score comparison table

```
| Criterion       | Round 1 | Round 2 | Round 3 | Delta (R1→R3) |
|-----------------|---------|---------|---------|----------------|
| Design Quality  | X/10    | X/10    | X/10    | +X             |
| Originality     | X/10    | X/10    | X/10    | +X             |
| Craft           | X/10    | X/10    | X/10    | +X             |
| Functionality   | X/10    | X/10    | X/10    | +X             |
| **Weighted Avg**| **X.X** | **X.X** | **X.X** | **+X.X**       |
```

### Frame navigation

List all frames with their IDs so the user can find them in Figma:
- "Round 1" — frame ID, position
- "Round 2" — frame ID, position
- "Round 3" — frame ID, position

### Key improvements across rounds

Summarize the most impactful changes that occurred, with before/after descriptions.

### Remaining issues

List the evaluator's Top 3 improvements from the final round — these are the remaining opportunities.

### Regression warnings

If any criterion scored LOWER in a later round, flag it with explanation.

---

## 6. Integration Notes

### Skill loading order

```
1. figma-use (MANDATORY — load first)
2. figma-generate-design (load for Round 1 full-screen builds)
3. frontend-design (load for aesthetic guidance in Planner)
4. design-loop criteria + calibration (loaded in initialization)
```

### MCP tools used

| Tool | Used by | Purpose |
|------|---------|---------|
| `use_figma` | Doer | All Figma canvas modifications |
| `get_metadata` | Doer only | Structural validation after edits |
| `get_screenshot` | Doer + Evaluator | Doer: visual validation during build. Evaluator: independent visual inspection |
| `search_design_system` | Initialization | Discover components, variables, styles |
| Agent (sub-agent) | Evaluator | Independent evaluation in separate context with `get_screenshot` access |

**Tool access boundaries**: The Evaluator sub-agent has access to `get_screenshot` only. It must NOT use `get_metadata` or `use_figma` — these reveal implementation details and create evaluation bias. The Evaluator receives the Figma file key and frame node ID so it can take its own screenshots.

### Project-level customization

To override evaluation criteria for a specific project:
1. Create `.claude/design-loop-criteria.md` in the project root
2. Follow the same format as [references/evaluation-criteria.md](references/evaluation-criteria.md)
3. The skill will automatically detect and use the project-level file

### Context window management

The loop generates significant context across rounds. To stay within limits:
- The Planner produces concise plans (not essays)
- The Doer uses targeted edits in Round 2+ (not full rebuilds)
- The Evaluator runs as a sub-agent (its full context doesn't consume the main window)
- Frame cloning preserves previous work without re-describing it

### Error recovery

- If a `use_figma` call fails: stop, read the error, fix the script, retry (per figma-use rules)
- If the Evaluator sub-agent fails to return structured output: re-run with stricter formatting instructions
- If frame cloning fails (node not found): re-read frame IDs from the canvas using `get_metadata`
- If a round produces worse scores than the previous: the Planner for the next round should analyze why and be more conservative
