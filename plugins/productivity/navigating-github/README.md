# Navigating GitHub

**Get set up on GitHub and learn it by doing — with an AI that adapts to your level.**

Use this skill to go from zero to a working GitHub setup, then learn git and GitHub through interactive hands-on exercises on your actual project. No slides, no docs, no assumed knowledge. Just real commands, real results, and explanations that make sense at your level.

## What It Does

### Get Set Up (the main event)

Say "set up my repo" or "help me with github" and the skill walks through everything:

1. **Connect to GitHub** — installs `gh` CLI, authenticates, configures git identity
2. **Initialize your repo** — `git init`, auto-generated `.gitignore`, first commit
3. **Create the remote** — `gh repo create`, push, shows the live URL

Skips anything already done. Complete beginners get full explanations ("GitHub is like Google Drive for code"). Experienced developers get it done in seconds.

### Learn by Doing

Say "teach me github" and start interactive lessons on your real project:

| Say This | What Happens |
|----------|-------------|
| "teach me github" | **GitHub 101** — make changes, commit, push. Guided step by step. |
| "what are branches" | **Branching Basics** — create a branch, make a change, switch back, see the difference. |
| "teach me PRs" | **Your First PR** — branch, push, create a PR, see the diff, merge it. |
| "teach me branching strategies" | **Branch Workflows** — feature branches, naming conventions, keeping branches current. |
| "teach me code review" | **PR Review Flow** — review a PR, leave comments, approve or request changes. |
| "teach me rebase" | **Rebase vs Merge** — interactive rebase, squash commits, clean history. |
| "teach me CI/CD" | **GitHub Actions** — write a workflow, push, watch it run. |

Every lesson follows **do-then-explain**: run the command, see what happens, then understand why. The AI checks understanding after each step before moving on.

### Daily Workflow (once you're set up)

| Say This | What Happens |
|----------|-------------|
| "save my work" | Stages changes safely, generates a commit message, commits |
| "share my changes" | Branches if on main, pushes, creates a PR |
| "what changed" | Translates `git status` and `git diff` into plain English |
| "I'm stuck" / "fix this" | Detects the problem and walks through the fix |

## Adaptive Skill Level

The skill figures out your level from your environment — commit message quality, branch usage, auth state — not from a questionnaire. It only asks when signals are genuinely ambiguous.

| Level | What You Experience |
|-------|-------------------|
| **Beginner** | Analogies, zero jargon, full step-by-step. The AI does it and teaches as it goes. |
| **Intermediate** | Light jargon with definitions, explains the "why." Handles it, asks you to confirm. |
| **Advanced** | Standard git vocabulary, brief rationale. Suggests, you decide. |
| **Expert** | Terse and technical. You drive, the AI assists. |

Continuously adapts — levels up when confidence grows, drops into teaching mode when confusion appears on any specific topic.

## Install

```bash
# Claude Code
/plugin marketplace add jeremylongshore/navigating-github

# Manual
git clone https://github.com/jeremylongshore/navigating-github.git
```

Works with: **Claude Code**, **Cursor**, **Windsurf**, **Aider**, **Continue** — any AI coding tool with terminal access.

## Safety (Non-Negotiable)

- Never pushes to `main`/`master` — always branches first
- Never force pushes without explicit confirmation
- Never commits secrets (`.env`, API keys, credentials)
- Never runs destructive operations without showing impact first

If you say "just push to main," it explains why branching is safer and branches anyway.

## Contributing

PRs welcome. If you see a gap in the curriculum, a confusing explanation, or a missing error recovery scenario — open an issue or submit a PR.

## License

MIT — see [LICENSE](LICENSE).

---

Built by [Jeremy Longshore](https://github.com/jeremylongshore) / [Intent Solutions](https://intentsolutions.io) for the [Tons of Skills](https://tonsofskills.com) marketplace.
