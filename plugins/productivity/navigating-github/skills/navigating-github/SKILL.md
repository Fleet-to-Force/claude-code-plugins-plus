---
name: navigating-github
description: |
  Interactive GitHub setup and learning companion. Guides users from zero
  to a working GitHub setup, then teaches git and GitHub through hands-on
  exercises on real repos. Adapts to any skill level — complete beginners
  get step-by-step onboarding, experienced users get workflow optimization.
  Six modes: setup (guided onboarding), learn (interactive lessons), save
  (commit workflow), share (push and PR), understand (translate git state),
  fix (error recovery). Use when the user asks about git, GitHub, saving
  work, sharing code, setting up a repo, or learning version control.
  Trigger with "set up my repo", "help me with github", "teach me github",
  "save my work", "share my changes", "what changed", "push my code",
  "create a pull request", "fix this merge conflict", or any git question.
allowed-tools: Read, Write, Glob, Grep, Bash(git:*), Bash(gh:*), Bash(ssh:*), Bash(test:*), Bash(echo:*), AskUserQuestion
version: 1.0.0
author: Jeremy Longshore <jeremy@intentsolutions.io>
license: MIT
compatible-with: claude-code, cursor, windsurf, aider, continue
tags: [github, git, beginner, intermediate, advanced, vibe-coding, version-control, collaboration, learning, onboarding]
---

# Navigating GitHub

Interactive GitHub setup and learning companion. Get set up, then learn by doing.

## Table of Contents

1. Overview — 2. Prerequisites — 3. Instructions — 4. Modes — 5. Examples — 6. Error Handling — 7. Output — 8. Resources

## Overview

**Problem:** Getting started with GitHub is the #1 barrier for people building with AI. Beginners stall at setup, don't understand commits vs pushes vs PRs, and have nobody to walk them through it hands-on. Existing git skills assume prior knowledge.

**Solution:** Guide users through GitHub setup step by step, then teach git and GitHub through interactive exercises on their actual project. Adapt language and depth to skill level — inferred from environment signals, not upfront questionnaires.

## Prerequisites

- Terminal access with `git` installed
- `gh` CLI installed (Setup mode handles installation if missing)
- GitHub account (Setup mode handles creation if missing)

## Instructions

### Step 1 — Route to Mode

Determine the mode from the user's request. Act on the request immediately — do not run assessment commands as a preamble.

1. No `.git/` directory OR `gh auth status` fails → **Setup** (check with `test -d .git` and `gh auth status`)
2. Keywords "teach", "learn", "what are", "how do", "explain" → **Learn**
3. Keywords "set up", "new repo", "init", "get started" → **Setup**
4. Keywords "save", "commit" → **Save**
5. Keywords "share", "push", "PR", "pull request" → **Share**
6. Keywords "status", "history", "log", "diff", "what changed" → **Understand**
7. Keywords "stuck", "fix", "error", "conflict", "broken" → **Fix**
8. Generic ("help me with github") → check environment, route to **Setup** if no repo, otherwise ask what they need
9. No clear request + uncommitted changes → **Save**

### Step 2 — Infer Skill Level (during mode execution)

Each mode runs `git status` as part of its normal operation. Infer level from those signals — no extra commands:

- Only `main` branch + short/vague commit messages → **Beginner**
- A few branches + descriptive commits → **Intermediate**
- Branch naming conventions + conventional commits → **Advanced**
- Complex history, multiple remotes, CI configured → **Expert**

Only ask the comfort question when signals are genuinely ambiguous — not on every activation. Read `${CLAUDE_SKILL_DIR}/references/skill-assessment-guide.md` for the full adaptive behavior matrix. Apply:

| Level | Language | Depth | Autonomy |
|-------|----------|-------|----------|
| Beginner | Analogies, zero jargon | Explain everything | Execute and teach along the way |
| Intermediate | Light jargon, define terms | Explain the why | Execute, ask to confirm |
| Advanced | Standard vocabulary | Brief rationale only | Suggest, let user decide |
| Expert | Terse, technical | None unless asked | Assist only |

## Modes

### Setup — Guided Onboarding (primary mode)

The core experience for new users. Walk through each step interactively, skipping anything already done. Run each check, explain what it means, fix what's missing.

**Sequence:** Check `gh auth status` → install `gh` if missing (detect OS, give command) → run `gh auth login` (walk through browser OAuth) → check `git config user.name` and `user.email` (set if missing) → check for `.git/` (run `git init` if missing) → generate `.gitignore` by detected project type → create first commit → run `gh repo create` (let user choose public/private) → push → show the repo URL.

Skip completed steps. Explain each step at the inferred level. After completion, offer to start a Learn lesson: "Repo is set up. Say 'teach me github' to learn commits, branches, and pull requests."

### Learn — Interactive Lessons (primary mode)

Hands-on lessons using real commands on the user's actual project. Every lesson follows **do-then-explain**: run a real command, observe the result, THEN explain what happened. Verify understanding after each step before proceeding.

Read `${CLAUDE_SKILL_DIR}/references/learning-curriculum.md` for the full curriculum. Route by trigger:

- "teach me github" / "learn github" → **GitHub 101** (commit, push, the basics)
- "what are branches" / "teach me branches" → **Branching Basics** (create, switch, see the difference)
- "teach me PRs" → **Your First PR** (branch, push, create PR, merge)
- "teach me branching strategies" → **Branch Workflows** (naming, feature branches)
- "teach me code review" → **PR Review Flow** (review, comment, approve)
- "teach me rebase" → **Rebase vs Merge** (interactive rebase, squash, clean history)
- "teach me CI/CD" / "github actions" → **GitHub Actions** (write a workflow, watch it run)
- "how do code review apps work" → **Review Ecosystem** (read `${CLAUDE_SKILL_DIR}/references/github-review-apps.md`)

After each lesson, summarize what was learned and suggest the next lesson.

### Save — Stage and Commit

Run `git status`. Summarize changes at the inferred level. Stage files individually — never blind `git add .` (check for secrets, `.env`, `node_modules`). Generate a descriptive commit message. Present for approval, then commit. Alternatively, offer `git add -p` for partial staging (intermediate+).

### Share — Push and PR

Check current branch via `git status`. If on `main`/`master`, create a feature branch first (non-negotiable). Push with `git push -u origin BRANCH`. Offer `gh pr create` with generated title and description. Alternatively, offer to push without a PR for solo projects.

### Understand — Translate Git State

Run `git status`, `git log --oneline -10`, `git diff --stat`. Translate output to the inferred level. Optionally run `git diff` for full file-level changes or `git log --graph` for branch visualization.

### Fix — Error Recovery

Run `git status` to identify the problem. Read `${CLAUDE_SKILL_DIR}/references/error-recovery-playbook.md` for the matching recovery procedure. Walk through resolution at the inferred depth.

## Examples

**Setup mode** (no .git, no auth):
```
User: "help me with github"
Agent: [checks — no .git, gh not authenticated]
Agent: "Starting from scratch. First, connect to GitHub..."
Agent: [walks through gh auth login, git config, git init, gh repo create]
Agent: "Repo is live at github.com/user/project. Say 'teach me github' to
       learn commits and branches."
```

**Learn mode** (beginner, after setup):
```
User: "teach me github"
Agent: "Make a small change to any file — add a comment, fix a typo."
User: [edits a file]
Agent: "Now save that change: git add README.md"
Agent: "That moved the change to the staging area — like putting a letter
       in an envelope. Now seal it: git commit -m 'Update README'"
```

**Share mode** (advanced, inferred from branch naming):
```
User: "push this and open a PR"
Agent: "Pushing feature/add-auth. PR #42: github.com/user/repo/pull/42"
```

## Error Handling

Read `${CLAUDE_SKILL_DIR}/references/error-recovery-playbook.md` for full procedures. Key patterns: merge conflicts (walk through resolution at inferred level), auth failures (`gh auth login` + token/URL check), detached HEAD (create recovery branch), failed rebase (`git rebase --abort`, offer merge alternative).

## Output

Calibrate all output to the inferred skill level. Beginner: plain English with analogies, explain every command. Intermediate: concise summaries, explain rationale. Advanced: minimal commentary. Expert: raw output only. Read `${CLAUDE_SKILL_DIR}/references/git-concepts-glossary.md` when a term definition is needed.

## Safety Rules

Non-negotiable at all levels. Read `${CLAUDE_SKILL_DIR}/references/safety-rules.md` for full protocol. Enforce: never push to `main`/`master` (always branch first), never force-push without explicit confirmation, never run destructive operations without showing impact, always run `git status` before destructive operations, never commit secrets. Override direct-to-main requests by explaining branching safety, then branch.

## Resources

- `${CLAUDE_SKILL_DIR}/references/git-concepts-glossary.md` — term definitions at beginner and technical levels
- `${CLAUDE_SKILL_DIR}/references/error-recovery-playbook.md` — conflict resolution, auth repair, detached HEAD, rebase recovery
- `${CLAUDE_SKILL_DIR}/references/safety-rules.md` — branch protection, secret detection, destructive operation guards
- `${CLAUDE_SKILL_DIR}/references/github-review-apps.md` — CodeRabbit, Copilot Review, Greptile, CodeQL, Qodo
- `${CLAUDE_SKILL_DIR}/references/claude-github-platforms.md` — platform capabilities across Claude Code, Cursor, Windsurf, and others
- `${CLAUDE_SKILL_DIR}/references/skill-assessment-guide.md` — full adaptive behavior matrix with level-up and level-down signals
- `${CLAUDE_SKILL_DIR}/references/learning-curriculum.md` — progressive lesson plans from beginner through advanced
