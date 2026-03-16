---
name: github-learn
description: Interactive GitHub setup and learning. Gets you set up on GitHub from scratch and teaches git through hands-on exercises on your actual project.
allowed-tools: Read, Write, Glob, Grep, Bash(git:*), Bash(gh:*), Bash(ssh:*), Bash(test:*), Bash(echo:*), AskUserQuestion
user-invocable: true
argument-hint: "[lesson]"
---

# /github-learn

Interactive GitHub setup and learning companion. Launch this command to get set up on GitHub or start a hands-on lesson.

## Instructions

### Step 1 — Determine Intent

Check the argument passed to the command:

- No argument → run environment check, route to **Setup** if not set up, otherwise offer lesson menu
- `setup` → route to **Setup**
- `101` or `basics` → route to GitHub 101 lesson
- `branches` → route to Branching Basics lesson
- `prs` → route to First PR lesson
- `workflows` → route to Branch Workflows lesson
- `review` → route to PR Review Flow lesson
- `rebase` → route to Rebase vs Merge lesson
- `actions` or `ci` → route to GitHub Actions lesson
- `apps` → route to Review Ecosystem lesson
- Any other text → treat as a question about git/GitHub, answer adaptively

### Step 2 — Environment Check (for Setup routing)

Run two checks:

```bash
gh auth status 2>&1
test -d .git && echo "GIT_REPO_EXISTS" || echo "NO_GIT_REPO"
```

If either fails, route to **Setup** regardless of what was requested — setup is a prerequisite.

### Step 3 — Infer Skill Level

Determine level from environment signals gathered in Step 2 and from `git status` output:

- Only `main` branch + short commit messages → **Beginner**
- A few branches + descriptive commits → **Intermediate**
- Branch naming conventions + conventional commits → **Advanced**
- Complex history, multiple remotes → **Expert**

Only ask the comfort question via `AskUserQuestion` when signals are genuinely ambiguous.

### Step 4 — Execute

**Setup:** Read and follow the Setup mode instructions in `${CLAUDE_SKILL_DIR}/skills/navigating-github/SKILL.md`. Walk through each prerequisite interactively, skip completed steps. After completion, show the lesson menu.

**Lesson menu** (when no specific lesson requested and setup is complete):

Present via `AskUserQuestion`:

> Pick a lesson (or say "teach me" followed by a topic):
>
> **Beginner**
> 1. GitHub 101 — commits, push, the basics
> 2. Branching Basics — create, switch, see the difference
> 3. First PR — branch, push, create a PR, merge
>
> **Intermediate**
> 4. Branch Workflows — naming, feature branches, staying current
> 5. PR Review Flow — review, comment, approve
> 6. Team Git — forks, upstream sync, collaboration
>
> **Advanced**
> 7. Rebase vs Merge — interactive rebase, squash, clean history
> 8. GitHub Actions — write a CI workflow, watch it run
> 9. Code Review Apps — CodeRabbit, Copilot Review, and the ecosystem

**Specific lesson:** Read `${CLAUDE_SKILL_DIR}/skills/navigating-github/references/learning-curriculum.md` and execute the matching lesson. Follow do-then-explain methodology: run real commands, observe results, explain afterward. Verify understanding after each step.

After each lesson, summarize what was learned and suggest the next lesson.

## Examples

```
/github-learn           → environment check → setup if needed, otherwise lesson menu
/github-learn setup     → guided GitHub setup from scratch
/github-learn 101       → GitHub 101 hands-on lesson
/github-learn branches  → Branching Basics lesson
/github-learn prs       → First PR lesson
/github-learn rebase    → Rebase vs Merge lesson
/github-learn actions   → GitHub Actions lesson
```
