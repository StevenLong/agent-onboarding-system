---
description: Bootstrap a minimal CLAUDE.md and the friction loop in this repo (interactive wizard)
argument-hint: (no arguments needed)
---

You are running an interactive setup wizard. Your job is to create a small starter set of agent onboarding docs in the current repository: a root `CLAUDE.md`, optional subdirectory `CLAUDE.md` files, a friction journal at `docs/CLAUDE-FRICTION.md`, a tribal-knowledge reference at `docs/CLAUDE-TRIBAL.md`, and four slash commands.

**Tone and pacing.** Be friendly and concise. Pause for the human at every marked decision point. Do not power through silently. Use plain language; assume the human may be a non-developer.

**No emojis** unless the human uses them first.

**Lean by default.** Recent research (Gloaguen et al., ETH Zurich, 2026; <https://arxiv.org/abs/2602.11988>) shows that LLM-generated context files harm coding-agent performance and that human-written ones offer only marginal gains, with cost increases of 20% or more. The dominant failure mode is duplicating what's already inferable from the code. This wizard's job is to produce the smallest useful `CLAUDE.md`, not the most thorough one.

**The rule for what earns a line: corrections, not descriptions.** A line earns its place if it corrects a training-bias default the model would otherwise get wrong, or surfaces non-obvious context the agent can't infer from the code. A description of what the code already says does not earn its place.

Follow the steps in order. Do not skip any. If something fails, stop and ask the human how to proceed instead of guessing.

---

## Step 0: Greet and confirm

Greet the human in one short paragraph. Tell them:

- What this wizard will do (create CLAUDE.md, optional subdir CLAUDE.mds, friction + tribal logs, four slash commands).
- That it will take 15 to 30 minutes, mostly waiting on their answers to a few questions.
- That nothing will be committed without their approval at the end.
- That the goal is a short, high-signal `CLAUDE.md` (target ~30 to 50 lines for the root). They can grow it later as patterns emerge from real use.

Then mention, briefly:

- If they're comfortable writing `CLAUDE.md` from scratch in 10 minutes, that's usually the better path. The wizard exists for cases where they want a baseline scaffold or are setting this up on a repo they don't know well.

Ask: "Ready to start? (yes / no / let me ask a question first)"

Wait for their answer. If they have questions, answer them and re-ask.

---

## Step 1: Sanity checks

Run these checks silently; only surface a question if a check fails or finds an existing file.

1. Confirm the working directory is a git repository (`git rev-parse --is-inside-work-tree`). If not, stop and tell the human to run the wizard from inside a git repo.
2. Confirm we are at the root of the repo (look for the parent of `.git`). If not, ask the human if they want to proceed at this path or move up.
3. Look for these files and note which exist:
   - `CLAUDE.md` (root)
   - `docs/CLAUDE-FRICTION.md`
   - `docs/CLAUDE-TRIBAL.md`
   - `.claude/commands/{friction,handoff,distill,tribal}.md`
   - `.claude/settings.json`
4. If any exist, present a list and ask: "I found existing files [list]. For each one I would normally create, do you want me to (a) overwrite, (b) skip and leave alone, or (c) abort the wizard? Default is **skip**." Wait for their answer per file or as a batch.

---

## Step 2: Gather facts (no prose)

The goal of this step is a short list of *verified facts* about the repo that you'll convert into corrections during the interview in Step 3. Do not generate prose descriptions of the codebase. Do not draft sentences for `CLAUDE.md` yet.

Run these commands and note the results:

1. **Stack identification.** Read `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, etc. (whichever are present). Note the language(s), build system, framework if obvious. Stop at facts; don't editorialize.
2. **Top-level layout.** `ls -d */` for the top-level directories. Note the 5 to 10 most active or largest.
3. **Recent activity.** `git log --oneline | head -20`. Note any recurring themes (e.g. "many commits about EDIEL," "recent activity in `app/billing/`"). Skip if no patterns are clear.
4. **Existing docs.** Note the presence of `README.md`, `CONTRIBUTING.md`, `docs/`, ADRs. Don't read them in detail; just note what exists.
5. **Tests.** Look for a test command in `package.json`, a `pytest.ini`, a `tests/` or `__tests__/` directory. Note the framework, or "no tests" if absent.

Output a short structured fact report (under 100 words):

```
Stack: <one line>
Layout: <list of top-level dirs>
Recent activity themes: <list, or "none clear">
Existing docs: <list>
Tests: <one line>
```

This report is internal scaffolding for the interview. Do not present it to the human as a draft of `CLAUDE.md`.

---

## Step 3: Interview the human (corrections, not descriptions)

Convert facts into corrections through targeted questions. Ask one question at a time. Follow up briefly when an answer warrants it; move on when it doesn't.

For each item in your fact report, ask a correction-shaped question. Examples:

- **Stack:** "I see this is a [stack X]. Have you ever had Claude reach for [common alternative the model would default to, e.g. webpack instead of Brunch]? If yes, that's worth a line in `CLAUDE.md`."
- **Layout:** "Is there a directory in this repo where Claude tends to make wrong assumptions? Examples: deprecated patterns it doesn't know about, module-specific quirks, files that look standard but have weird behavior."
- **Recent activity:** "Are there areas of active refactoring or migration where the codebase changes faster than docs would catch up?"
- **Tests:** If no tests, "Do you want Claude to know it can't verify changes by running tests?" If tests exist, ask whether there's anything non-obvious about the test setup; otherwise skip.

Avoid:

- "Tell me everything about X." (Too broad. The human will freeze.)
- "What does this codebase do?" (Description, not correction.)
- Questions that produce a yes/no without a useful follow-up.

Good signals an answer is worth recording:

- Specific files or functions named.
- A specific wrong assumption corrected.
- A trap or gotcha that isn't visible from reading code.
- A library or pattern the team has rejected, with reason.

Skip the question or note "no entry needed" if the answer is generic, vague, or describes something the model could find on its own.

Aim for 3 to 8 corrections from the interview. More than that and the file is bloating; fewer and you may have under-asked.

After the topic-driven questions, also ask:

- "Are there subdirectories in this repo that have their own quirks worth a separate `CLAUDE.md`? Examples: a billing module with weird state machines, a migration-in-progress area, a high-quirk legacy directory. The default is **none**; only add subdir docs if a directory specifically generates friction the root file can't cover."

If yes, note which subdirectories. You'll handle them in Step 5.

---

## Step 4: Draft the root CLAUDE.md

Using the corrections from Step 3 plus the boilerplate meta sections at the bottom of this file, draft a root `CLAUDE.md` with these sections, in this order:

1. **Title and one-line purpose.**
2. **Stack:** a tight bullet list including only stack assertions that correct training-bias defaults (e.g., "Brunch, not webpack" if Brunch is in use). Do not list every dependency. List only those where the agent would otherwise default wrong.
3. **Run** (only if there's a non-obvious setup): how to run the project locally. If `npm start` or `docker compose up` works as expected, omit this section.
4. **Tests** or **No tests.** If there are no tests, include the directive that the agent cannot claim "verified" without manual exercise. If tests exist, mention the command in one line.
5. **Quirks & landmines:** the corrections from Step 3 about non-obvious traps, one bullet per correction.
6. **Conventions:** training-bias-correcting conventions only (e.g., "HTML5 mode is off, URLs use `#/...`" if true). Skip anything inferable from one example file.
7. **Subdirectory CLAUDE.md files** (if any from Step 3): one bullet per subdir with its topic.
8. **Friction & tribal knowledge:** paste the boilerplate text from below verbatim.
9. **Session continuity:** paste the boilerplate text from below verbatim.
10. **Precedence:** paste the boilerplate text from below verbatim.

**Length budget: 30 to 50 lines for a typical repo.** If the draft exceeds 75 lines, prune.

**Pruning criteria.** For each candidate line, ask:

- Could the agent infer this from reading the code? If yes, **cut**.
- Is this a correction the agent would otherwise get wrong, or non-obvious context it can't infer? If no, **cut**.

Be ruthless. The default is "leave it out."

**Style:** terse, no emojis, no en or em dashes (use periods, commas, colons, parentheses, or rephrase). Hyphens in compound words and lists are fine.

Show the draft to the human and ask: "Does this look right? You can (a) accept as-is, (b) tell me what to change, or (c) cancel the wizard."

If they accept, write the file to `CLAUDE.md` at the repo root.

If they ask for changes, iterate until they accept.

---

## Step 5: Subdirectory CLAUDE.mds (only if the human chose to include them)

For each subdirectory the human approved:

1. Run a focused fact-gather, scoped to that subtree: `ls`, summary of file count, recent activity (`git log -- <subdir> | head -20`). Do not write prose summaries.
2. Ask the human: "What corrections does Claude need for this subdirectory? What does it tend to get wrong here? What integrations or external systems does it touch with non-obvious gotchas?"
3. Capture concrete answers as corrections, one bullet per item.
4. Draft a subdirectory `CLAUDE.md` with: title + one-line purpose, the corrections, and a short "Quirks and gotchas" or "External integrations" section if relevant. Skip the boilerplate sections (Friction & tribal, Session continuity, Precedence); they live in the root.
5. **Length budget: 30 to 80 lines** per subdirectory.
6. Same pruning criteria as Step 4.
7. Show the draft, get human approval, then write to `<subdir>/CLAUDE.md`.

Repeat for each chosen subdir. Save each as you go; do not batch.

---

## Step 6: Create the boilerplate files

Write these files. Do not ask the human about their contents; they are always the same.

### 6a. `docs/CLAUDE-FRICTION.md`

If `docs/` does not exist, create it. Then write this content (verbatim):

```markdown
# CLAUDE-FRICTION

A running log of friction, gaps, and wrong assumptions encountered while working with the `CLAUDE.md` family of docs. Both humans and AI agents append here. Periodically someone runs `/distill`, which proposes edits to the actual docs based on patterns in the log.

This file is **not auto-loaded** into agent sessions and is **not authoritative**. It is a journal, not a rulebook.

## How to log an entry

**Humans:** run `/friction <one or two sentences>`. The agent appends a properly-formatted, dated entry on your behalf.

**AI agents:** append directly to the "Entries" section as soon as you notice friction.

## How to write an entry

- **Be concrete, not interpretive.** "Suggested webpack; user pointed out the project uses Brunch" beats "Claude was weird about bundlers."
- **One or two sentences is plenty.** This is a journal, not a report.
- **Don't try to fix the docs in the entry.** Describe the friction; let `/distill` propose the fix.

## Entries

<!-- Format: ### YYYY-MM-DD [tag] - short title
     One or two sentences describing what happened.
     Author: <git user.name, or "human", or "agent">
-->

_(none yet)_
```

### 6b. `.claude/commands/friction.md`

If `.claude/commands/` does not exist, create it. Then write this content (verbatim):

```markdown
---
description: Log a friction point or doc gap to docs/CLAUDE-FRICTION.md
argument-hint: what happened (one or two sentences)
---

The user is recording human-flagged friction about the CLAUDE.md docs in this repo. Their description follows on the next line.

$ARGUMENTS

## Step 1: Get the author identity

Run \`git config user.name\`. Fall back to \`git config user.email\`. If both are empty, use \`human\`.

## Step 2: Pick a tag

A short lowercase token grouping entries by area or kind. Common tags:

- A **module name** for localized friction: \`[billing]\`, \`[hubs]\`, \`[feeds]\`, etc.
- A **kind** for cross-cutting: \`[stack]\`, \`[convention]\`, \`[doc-error]\`, \`[meta]\`.

Reuse existing tags from the log when possible. Don't agonize.

## Step 3: Append the entry

Append to the bottom of \`docs/CLAUDE-FRICTION.md\`. Remove the \`_(none yet)_\` placeholder if present. Format:

\`\`\`
### YYYY-MM-DD [tag] - <short title>
<one or two sentences>
Author: <name>
\`\`\`

## Step 4: Stop

Don't propose doc edits. Distillation is `/distill`'s job.
```

### 6c. `.claude/commands/handoff.md`

Write this content (verbatim):

```markdown
---
description: Snapshot the current session state to HANDOFF.md so a fresh session can pick up the thread
argument-hint: optional note about what to emphasize
---

The user is taking a checkpoint of the current session before pausing, restarting, or context-switching. Write (or overwrite) `HANDOFF.md` at the repo root with a tight snapshot of where the session is.

If the description below is non-empty, weight it heavily; the user is telling you what they care about preserving.

$ARGUMENTS

Write `HANDOFF.md` with this exact structure:

\`\`\`
# HANDOFF

_Snapshot taken YYYY-MM-DD HH:MM. Personal session state; this file is gitignored. Overwrite each time, do not accumulate history._

## Where we are

<One or two sentences. What task, what phase.>

## Decided this session

<Bullet list of decisions or commitments made in the conversation, with one-line rationale where it isn't obvious. Omit the section if nothing notable was decided.>

## Pending

<Bullet list of concrete next steps: file paths, specific actions. Avoid vague items like "continue the work".>

## Open questions

<Bullet list of things raised but not resolved. Omit if none.>

## Watch out for

<Bullet list of context easy to miss on restart: bugs being worked around, things tried and rejected, fragile state, non-obvious assumptions. Omit if none.>
\`\`\`

Use today's date and the current time. Total length: 20 to 50 lines. Be concrete, not narrative. Overwrite any existing `HANDOFF.md`; do not append.

Then stop. Do not propose other doc edits, do not read unrelated files, do not start follow-up work. The handoff is a snapshot, not an action plan.
```

### 6d. `.gitignore`

Append a `HANDOFF.md` entry. If `.gitignore` does not exist, create it with just that one line. If it exists, append (do not duplicate; check for an existing match first):

```
# Personal session-handoff snapshots (per-developer, not team artifact)
HANDOFF.md
```

### 6e. `.claude/commands/distill.md`

Write this content (verbatim):

```markdown
---
description: Review docs/CLAUDE-FRICTION.md for patterns and propose edits to the actual docs
argument-hint: optional focus (e.g. "billing area only" or "last month")
---

The user is asking you to review the accumulated friction log and propose distillation: turning recurring patterns or high-signal entries into actual edits to the CLAUDE.md family of docs.

If the user provided a focus, weight it heavily.

$ARGUMENTS

## Step 1: Read the inputs

Read \`docs/CLAUDE-FRICTION.md\`. Also read the root \`CLAUDE.md\` and any subdirectory \`CLAUDE.md\` files relevant to the entries.

If the log is empty (only the placeholder \`_(none yet)_\` remains), say so and stop.

## Step 2: Identify candidates for distillation

A candidate is one of:

- **Pattern:** three or more entries on the same theme. Patterns earn doc edits.
- **Factual error:** a single entry pointing at something in the docs that is wrong (renamed file, removed function, stale claim). Factual errors earn doc edits even as singletons.

A non-candidate (do not propose action):

- A single entry about one-off friction. One report is not yet a pattern.
- An entry that's a question or vent rather than a concrete observation.
- An entry already addressed in a prior \`## Distilled\` section.

If there are no candidates, say so plainly and stop. Don't invent work.

## Step 3: For each candidate, draft a proposal

\`\`\`
PROPOSAL N
Cluster/entry: <list the entries by date and one-line summary>
Theme: <one sentence>
Proposed edit: <which file, what specific addition or correction>
Risk: <one line; "low" is acceptable>
\`\`\`

Don't write the actual edit yet. Just the proposal.

## Step 4: Present and confirm

Show all proposals as a numbered list. Ask: "Which of these should I apply? (e.g. 'all', '1 and 3', 'none')"

Wait for the answer. If they want to clarify or revise a specific proposal, do so before applying.

## Step 5: Apply approved edits

For each approved proposal:

1. Make the actual edit to the named file. Prefer Edit over Write unless adding a wholly new section.
2. Show the diff before saving when the change is non-trivial.
3. If two approved edits would conflict, apply sequentially and ask the user how to reconcile.

## Step 6: Mark the log

Append a new section to \`docs/CLAUDE-FRICTION.md\` at the very bottom:

\`\`\`
## Distilled YYYY-MM-DD

- **<theme>** -> <file>: <one-line summary of the edit>. Addresses entries dated [list].
\`\`\`

Use today's date. Do not delete the original entries.

## Step 7: Wrap up

Tell the user briefly: how many proposals, how many applied, how many entries absorbed, where the edits landed. Then stop.
```

### 6f. `.claude/commands/tribal.md`

Write this content (verbatim):

```markdown
---
description: Run a structured interview to extract tribal knowledge into docs/CLAUDE-TRIBAL.md
argument-hint: optional focus (e.g. "billing", "the auth flow", "things you'd warn a new hire about")
---

The user wants to do a tribal-knowledge interview. Your job is to extract specific, undocumented context from their head and propose additions to \`docs/CLAUDE-TRIBAL.md\`.

If they provided a focus, weight it heavily.

$ARGUMENTS

## How to interview

Ask one question at a time. Wait for an answer before the next question.

Start with one of these openers:

- "What's something in this codebase you understand that you suspect nobody else does?"
- "What's a trap you've fallen in here that you'd warn a new teammate about?"
- "Is there code that looks fine on the surface but does the wrong thing in some specific scenario?"
- "What's something you know about this system that isn't written down anywhere?"
- "What's the last weird thing that surprised you in this code, that you wish you'd known earlier?"

Listen, then follow up. Good follow-ups: "Why is it that way?", "What happens if someone changes it without knowing?", "Has this caused a bug before?", "Is there a specific file or function I should record?"

Avoid "Tell me everything about X" or "Are there any other quirks?"; both freeze people up.

Keep going until the user signals they're done. Don't push past genuine recall. Total length: 5 to 15 minutes typical.

## Capturing the knowledge

After the interview, draft entries for \`docs/CLAUDE-TRIBAL.md\`. Each entry should be:

- **Specific**, not abstract. Names of files, functions, or behaviors.
- **Concrete**, not narrative.
- **Attributed.** End with \`(per <name>, YYYY-MM-DD)\`. Get the name from \`git config user.name\` (fall back to \`git config user.email\` if empty). If both fail, ask the user briefly. Substitute the actual name.

Show the draft entries to the user. Ask: "Do these capture what you said? Want to revise wording or add anything?"

Iterate until they're happy.

## Writing to the file

If \`docs/CLAUDE-TRIBAL.md\` doesn't exist, create it with the template at the bottom of this command.

Append the new entries at the end of existing entries (after the \`## Entries\` heading). If the placeholder \`_(none yet)_\` is still there, remove it as you add the first real entry.

## Wrap up

Tell the user how many entries were added and that \`/tribal\` can be re-run later. Then stop.

---

## Template for new `docs/CLAUDE-TRIBAL.md` (use only when creating)

\`\`\`markdown
# CLAUDE-TRIBAL

Specific, anecdotal, hard-won knowledge about this codebase that isn't visible from reading the code itself: traps, gotchas, "we tried that and stopped because Y," "ask Z before changing this," and similar context that lives mostly in people's heads.

This file complements \`CLAUDE.md\`. Where \`CLAUDE.md\` captures conventions and structure (visible in code), this captures specific decisions and warnings (not visible).

## Entries

<!-- Format: ### <topic or location>
     The thing to know.
     (per <name>, YYYY-MM-DD)
-->

_(none yet)_
\`\`\`
```

### 6g. `docs/CLAUDE-TRIBAL.md`

Use the same template embedded inside Step 6f above. If `docs/` does not exist, create it. Write the file (the template content, not wrapped in extra fences).

### 6h. `.claude/settings.json` (only if it does not already exist)

Write the file with content:

```json
{}
```

Do not add hooks. Do not add permissions. The empty object is a placeholder so the file exists for future config.

### 6i. `README.md` (discoverability section)

If a `README.md` exists at the repo root, append the section below to the **end** of the file (after a blank line). If no `README.md` exists, ask the user whether to create a minimal one with just this section, or skip.

The section to append (verbatim, but substitute the actual subdirectory `CLAUDE.md` paths into the relevant bullet if any subdir docs were created in Step 5; if none were, omit that bullet entirely):

```markdown
## AI agent onboarding

This repo has agent onboarding docs that give AI coding assistants (such as Claude Code) project context up front. Key files:

- `CLAUDE.md` (root): stack, conventions, quirks. Auto-loaded by Claude Code each session.
- Subdirectory `CLAUDE.md` files (`<paths>`): deeper context for high-quirk areas, auto-loaded only when an agent works in that subtree.
- `docs/CLAUDE-FRICTION.md`: a journal of friction with the docs. Improves over time.
- `docs/CLAUDE-TRIBAL.md`: hard-won knowledge that isn't visible from reading the code (traps, "ask X before changing Y," and the like). Starts empty; grows over time.

Slash commands available inside Claude Code in this repo:

- `/friction <description>`: log a doc gap.
- `/handoff <optional note>`: snapshot session state before a pause or restart.
- `/distill`: review accumulated friction and propose doc edits.
- `/tribal <optional focus>`: extract undocumented knowledge through a structured interview.
```

After appending, mention to the user that they can move the section higher in the README if they want more visibility. Default placement is the end (safest, doesn't disrupt existing structure).

---

## Step 7: Final review and propose commit

Show the human:

1. A list of all files created or modified, with their sizes (line counts).
2. A short proposed git commit message, e.g.:
   ```
   Add CLAUDE.md and friction infrastructure for agent onboarding

   - Root CLAUDE.md (corrections-not-descriptions; ~30-50 lines)
   - <N> subdirectory CLAUDE.md files for high-quirk areas
   - docs/CLAUDE-FRICTION.md for accumulating friction log
   - docs/CLAUDE-TRIBAL.md for hard-won knowledge (initially empty)
   - /friction, /handoff, /distill, /tribal slash commands
   - HANDOFF.md added to .gitignore (personal session state)
   - README.md updated with an "AI agent onboarding" section
   ```

Ask: "Ready to commit? (yes / no / let me review the files first)"

If yes: run `git add` for the new files (use specific paths, never `git add .`) and `git commit` with the proposed message. Do not push.

If no or let me review: leave the files in the working tree and tell the human they can review and commit later.

---

## Step 8: Wrap up

Tell the human, in one short paragraph:

- What was created (one line).
- The four slash commands now available:
    - `/friction <description>` logs ad hoc doc gaps.
    - `/handoff <optional note>` snapshots session state before a pause or restart.
    - `/distill` reviews accumulated friction and proposes doc edits.
    - `/tribal <optional focus>` runs a structured interview to capture undocumented knowledge.
- That they can delete `.claude/commands/init-agent-docs.md` if they don't plan to re-run the wizard. (The other files stay.)
- That the docs are deliberately small at the start. The friction log and `/distill` are the mechanism for growing them in the right places, only when real use shows the gap.

End the session.

---

## Boilerplate content (use verbatim in Step 4)

### Friction & tribal knowledge section (paste into root CLAUDE.md)

```markdown
## Friction & tribal knowledge

If you hit doc friction during a session, append a one-liner to `docs/CLAUDE-FRICTION.md` (or use `/friction`). Don't edit `CLAUDE.md` mid-session; that's `/distill`'s job.

Hard-won, not-in-the-code context (traps, "we tried that and stopped because Y") goes in `docs/CLAUDE-TRIBAL.md`. Scan it for entries that mention any file or area you're about to touch. Use `/tribal` to add structured entries.

The friction log is **not authoritative**. It's raw observation, used as input to distillation later.
```

### Session continuity section (paste into root CLAUDE.md)

```markdown
## Session continuity

If `HANDOFF.md` exists at the repo root, read it at the start of the session and orient the user in one sentence. The file is gitignored.

When the user is winding down, suggest `/handoff` to capture state.
```

### Precedence section (paste into root CLAUDE.md)

```markdown
## Precedence

If two docs contradict, the more specific one wins:

1. A topic-specific doc referenced for the current task.
2. A subdirectory `CLAUDE.md` overrides this root.
3. This root is the baseline.

Log contradictions to `docs/CLAUDE-FRICTION.md`.
```
