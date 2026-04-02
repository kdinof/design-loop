# Design Loop

Reusable infrastructure for iterative Figma design creation using the Generator-Evaluator harness pattern (Planner → Doer → Evaluator loop with frame versioning).

## Language Rules

- All skills, prompts, evaluation criteria, code, and technical documentation MUST be written in English
- User-facing communication (chat responses, summaries, questions) should be in the user's preferred language — configure below
- Never mix languages within a single file — English only for all artifacts

**User language**: Russian  <!-- Change this to your language (e.g., English, Spanish, Chinese) -->

## Project Structure

```
.claude/skills/design-loop/
├── SKILL.md                          # Main orchestrator
└── references/
    ├── evaluation-criteria.md        # 4-point scoring rubric
    └── calibration-examples.md       # Few-shot evaluator calibration
```

## Key Dependencies

- `figma-use` skill — mandatory prerequisite for every `use_figma` call
- `figma-generate-design` skill — for full-screen builds in Round 1
- `frontend-design` skill — aesthetic guidance for Planner
- Figma MCP server — must be connected

## Architecture

- **Planner** (inline): produces actionable design plan from brief + evaluator feedback
- **Doer** (inline): executes plan in Figma via `use_figma`, validates with `get_metadata` + `get_screenshot`
- **Evaluator** (sub-agent): independent critic with no access to plan/code, scores design against rubric
- Each round clones the previous frame — user sees full evolution side by side
