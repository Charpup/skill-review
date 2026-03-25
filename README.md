# skill-review

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-blueviolet?logo=anthropic&logoColor=white)](https://github.com/Charpup/skill-review)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![skill-vetter](https://img.shields.io/badge/dep-skill--vetter-blue?logo=github)](https://github.com/clawdbot/skills)
[![value-first-gate](https://img.shields.io/badge/dep-value--first--gate-blue?logo=github)](https://github.com/Charpup/value-first-gate)
[![inbox-triage](https://img.shields.io/badge/dep-inbox--triage-blue?logo=github)](https://github.com/Charpup/inbox-triage)
[![Pure Prompt](https://img.shields.io/badge/type-Pure_Prompt-green)](https://github.com/Charpup/skill-review)
[![Risk Level](https://img.shields.io/badge/risk-🟢_LOW-brightgreen)](https://github.com/Charpup/skill-review)

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

## Dependencies

This skill orchestrates three other skills. Install them before using `skill-review`:

| Skill | Repo | Required for |
|-------|------|-------------|
| [`skill-vetter`](https://github.com/clawdbot/skills) | `clawdbot/skills` | Phase 2 — Security audit |
| [`value-first-gate`](https://github.com/Charpup/value-first-gate) | `Charpup/value-first-gate` | Phase 4 — Value evaluation |
| [`inbox-triage`](https://github.com/Charpup/inbox-triage) | `Charpup/inbox-triage` | Phase 7 — Inbox cleanup |

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

```bash
git clone https://github.com/Charpup/skill-review.git ~/.claude/skills/skill-review
```

Or copy `SKILL.md` and `_meta.json` to `~/.claude/skills/skill-review/`.

## License

MIT
