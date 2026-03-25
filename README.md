# skill-review

A Claude Code skill that orchestrates the full lifecycle of evaluating and deploying new skills — from security audit to deployment decision to inbox cleanup.

## What it does

When a new `SKILL.md` appears in `inbox/` (or you want to evaluate any skill), this skill runs a complete 7-phase pipeline automatically:

```
Phase 1: Locate   → Explore subagent reads the skill file
Phase 2: Audit    → skill-vetter security check (risk gate)
Phase 3: Analyze  → 4-dimension analysis (intent / quality / executability / fit)
Phase 4: Evaluate → value-first-gate (first-principles challenge, 6-dimension score)
Phase 5: Decide   → GO / REVISE / NO-GO with consolidated reasoning
Phase 6: Execute  → Install to ~/.claude/skills/ OR archive + append value-review.md
Phase 7: Cleanup  → inbox-triage removes processed files
```

## Prerequisites

The following skills must be installed for full functionality:

| Skill | Required for |
|-------|-------------|
| [`skill-vetter`](https://github.com/clawdbot/skills) | Phase 2 — Security audit |
| [`value-first-gate`](https://github.com/Charpup/skill-value-first-gate) | Phase 4 — Value evaluation |
| `inbox-triage` | Phase 7 — Inbox cleanup |

Graceful degradation is built in — if a sub-skill is unavailable, the phase falls back to an inline checklist.

## Trigger phrases

- `评估这个 skill` / `这个 skill 值得装吗`
- `vet this skill` / `review this skill`
- `inbox 有新 skill` / `help me evaluate a skill`
- Any mention of a new SKILL.md that needs evaluation

## Output

Every run appends a structured record to `value-review.md` and outputs a summary card:

```
## Skill Review 完成

| Item | Result |
|------|--------|
| Skill | example-skill |
| Security | 🟢 LOW |
| value-first-gate | GO (24/30) |
| Fit | ★★★★☆ |
| Decision | ✅ Installed |
| value-review.md | ✅ Appended |
| Inbox | ✅ Cleaned |
```

## Architecture note

- **Subagent usage**: Phase 1 uses an Explore subagent for isolated file reading. All other phases run sequentially (each depends on the previous phase's output).
- **Pure Prompt skill**: No executable code, no external API calls, no network access. Risk level: 🟢 LOW.

## Installation

Copy `SKILL.md` and `_meta.json` to `~/.claude/skills/skill-review/`.

Or clone this repo:

```bash
git clone https://github.com/Charpup/skill-review.git ~/.claude/skills/skill-review
```

## License

MIT
