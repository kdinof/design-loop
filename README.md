# Design Loop

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for iterative Figma design creation using the **Generator-Evaluator harness pattern**. It runs a Planner → Doer → Evaluator loop for N rounds, cloning the frame each round so you see the full design evolution side by side.

Based on Anthropic's research into [building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — specifically the insight that models confidently praise their own work even when it's mediocre. Separating the evaluator into an independent context window is the fix.

## How it works

```
┌─────────────────────────────────────────────────────────────┐
│                     Design Loop                             │
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────────┐   │
│  │ PLANNER  │───▶│  DOER    │───▶│  EVALUATOR           │   │
│  │ (inline) │    │ (inline) │    │  (independent agent)  │   │
│  └──────────┘    └──────────┘    └──────────────────────┘   │
│       ▲                                    │                │
│       └────────── feedback ◀───────────────┘                │
│                                                             │
│  Round 1          Round 2          Round 3                   │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐               │
│  │ Frame 1 │     │ Frame 2 │     │ Frame 3 │               │
│  │         │     │ (clone) │     │ (clone) │               │
│  │ 4.2/10  │     │ 6.1/10  │     │ 7.8/10  │               │
│  └─────────┘     └─────────┘     └─────────┘               │
└─────────────────────────────────────────────────────────────┘
```

Each round:
1. **Planner** creates an actionable design plan from the brief + previous evaluator feedback
2. **Frame Versioning** clones the previous frame (Round 1 creates a new one)
3. **Doer** executes the plan in Figma, validates with screenshots after each section
4. **Annotation** creates an in-Figma changelog below each frame (what changed + why)
5. **Evaluator** (separate sub-agent) scores the design against a 4-criterion rubric
6. **Loop Control** decides whether to continue (score < 9.0) or finish

The evaluator only sees screenshots — no code, no node structure, no plan. This prevents self-evaluation bias.

## Evaluation Criteria

Designs are scored on 4 criteria (1-10 scale):

| Criterion | Weight | What it measures |
|-----------|--------|-----------------|
| **Design Quality** | 2x | Coherence, unified visual language, emotional direction |
| **Originality** | 2x | Custom decisions vs. defaults/templates, AI slop detection |
| **Craft** | 1x | Technical precision — spacing, alignment, typography hierarchy |
| **Functionality** | 1x | Usability — clear purpose, obvious CTAs, information flow |

Design Quality and Originality carry double weight because AI excels at technical execution but struggles with cohesive vision and template avoidance.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI or desktop app
- [Figma Console MCP server](https://github.com/southleft/figma-console-mcp) — connected and running
- A Figma file open in the desktop app

### Required Claude Code skills

The following skills must be available in your Claude Code environment:

- **`figma-use`** (mandatory) — prerequisite for every Figma MCP call
- **`figma-generate-design`** (recommended) — for full-screen builds in Round 1
- **`frontend-design`** (optional) — aesthetic guidance to avoid generic AI outputs

## Installation

1. Clone the repo:
   ```bash
   git clone https://github.com/gaynutdinov-k/design-loop.git
   ```

2. Copy the skill to your project or global Claude Code skills directory:
   ```bash
   # Project-level (recommended)
   cp -r design-loop/.claude/skills/design-loop your-project/.claude/skills/

   # Or global level
   cp -r design-loop/.claude/skills/design-loop ~/.claude/skills/
   ```

3. Copy `CLAUDE.md` to your project root (or merge with your existing one):
   ```bash
   cp design-loop/CLAUDE.md your-project/CLAUDE.md
   ```

4. Make sure the Figma Console MCP server is configured in your Claude Code settings.

## Usage

Invoke the skill in Claude Code:

```
/design-loop Create a SaaS pricing page with 3 tiers, dark theme, bold typography
```

### Arguments

| Argument | Required | Default | Description |
|----------|----------|---------|-------------|
| `brief` | Yes | — | Design task description |
| `--rounds N` | No | 3 | Number of iteration rounds (1–5) |
| `--criteria-override path` | No | — | Path to custom evaluation criteria file |

### Examples

```
/design-loop Landing page for a fintech app with trust-building design

/design-loop --rounds 5 Portfolio website for a creative agency, minimal black-and-white aesthetic

/design-loop --criteria-override .claude/my-criteria.md Mobile banking dashboard
```

### First run — Design System setup

On the first launch, the skill will ask for your Figma design system URL. It analyzes your DS file (components, variables, styles, typography) and saves the analysis for all subsequent rounds. If you don't have a design system, it will work without one.

## Customization

### Evaluation criteria

Override the default evaluation rubric by creating `.claude/design-loop-criteria.md` in your project root. Follow the format in [references/evaluation-criteria.md](.claude/skills/design-loop/references/evaluation-criteria.md).

### Language

The skill communicates with the user in the language configured in `CLAUDE.md`. Default is Russian — change the `User language` line to your preferred language.

### Calibration examples

The evaluator uses [calibration examples](.claude/skills/design-loop/references/calibration-examples.md) to anchor its judgment. You can modify these to match your design standards.

## Project Structure

```
.claude/skills/design-loop/
├── SKILL.md                          # Main orchestrator (all phases)
└── references/
    ├── evaluation-criteria.md        # 4-criterion scoring rubric
    └── calibration-examples.md       # Few-shot evaluator calibration
```

## How the evaluator stays honest

The key architectural decision: the Evaluator runs as a **separate sub-agent** in its own context window. It receives:
- The Figma file key and frame node ID (to take its own screenshots)
- The design brief
- Evaluation criteria + calibration examples
- Design system reference (colors, fonts, spacing — for DS conformance checks)

It does **not** receive:
- The Planner's plan
- The Doer's code
- Node structure or metadata

This isolation prevents the "I made it so it must be good" bias that plagues single-context evaluation.

## License

MIT
