# Agent Onboarding System

A lightweight context-persistence system for AI coding assistants working on real codebases.

## The problem

AI coding assistants start cold. Every session, they know nothing about the project beyond what is directly in front of them: file names, the current file, the prompt just typed. No conventions. No rejected approaches. No awareness of what is mid-refactor or quietly broken. The user spends the first part of every session re-explaining things they have already explained many times before.

Worse, the AI's defaults are drawn from training data dominated by current best practice. In a repository using an older stack, it will reflexively suggest modern alternatives unless told otherwise. In a repository with custom conventions, it will produce something that looks idiomatic but is wrong.

This is fixable, if the project context travels with the repository.

## The solution

Four connected pieces, all committed to the repository:

1. **A `CLAUDE.md` file** at the repository root. Auto-loaded at the start of every Claude Code session. Contains what an agent needs to know up front: stack, conventions, quirks, what not to suggest. Optional subdirectory `CLAUDE.md` files auto-load only when an agent works in that area.

2. **A feedback log** (`docs/CLAUDE-FRICTION.md`) and a **tribal knowledge reference** (`docs/CLAUDE-TRIBAL.md`). The feedback log is a running record of friction the docs caused or failed to prevent. The tribal knowledge reference captures context that is not visible from reading the code: why a workaround exists, what was tried and abandoned, what to ask before changing something. Periodic distillation turns the feedback log into actual edits to `CLAUDE.md`. The docs improve over time instead of decaying.

3. **Four slash commands** for cheap maintenance:
   - `/friction` logs a doc gap in one line
   - `/handoff` snapshots live session state before a pause or restart
   - `/distill` reviews the feedback log and proposes edits to `CLAUDE.md`, with human approval
   - `/tribal` runs a structured interview to capture undocumented context

4. **A discoverability section** added to the repo's `README.md` so the system can be found without already knowing it exists.

Setup is a single slash command (`/init-agent-docs`) that runs inside Claude Code and walks through guided questions. Roughly 15 to 30 minutes per repo.

## What this gives you

- A new Claude session opens with a working understanding of the codebase. Less re-explaining, more getting to the work.
- The AI stops reflexively suggesting approaches that do not fit your codebase. When it does suggest one, the suggestion is at least informed by why the code is the way it is.
- Friction is captured cheaply instead of being lost. Over weeks, the log shows exactly where the docs are weak.
- Sessions do not have to start from zero after a restart. `/handoff` records live state so picking up costs less.

## Installation

1. Copy `init-agent-docs.md` into your repository's `.claude/commands/` folder.
2. Open Claude Code in that repository.
3. Run `/init-agent-docs`.
4. Answer a handful of questions about your project.
5. Review and commit what the wizard creates.

That is it. The wizard handles the rest.

## Design principles

**Corrections, not descriptions.** A line earns its place in `CLAUDE.md` if it corrects a default the model would otherwise get wrong, or surfaces context the agent cannot infer from the code. A description of what the code already says does not earn its place.

**Small by default.** The target for a root `CLAUDE.md` is 30 to 50 lines. The friction log and `/distill` are the mechanism for growing it, only where real use shows the gap.

**Travels with the repository.** Everything is plain text files committed to the repo. No external services, no central platform, no licensing.

## Limitations

- The full system (slash commands, wizard) works inside Claude Code. Other AI tools may read `CLAUDE.md` as plain text but will not run the commands.
- The docs are only as good as what the wizard finds plus what humans add. Context that lives entirely in someone's head still has to be elicited and written down.
- This is version 0.something. The shape is settled; details will move.

## License

MIT
