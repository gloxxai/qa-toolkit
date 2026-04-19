---
name: workflow-setup
description: Scaffold a three-file Claude workflow (CLAUDE.md + CONTEXT.md + REFERENCES.md) at the start of any new project — a code repo, an Obsidian folder, a content project, anywhere Claude will be working repeatedly. Use when the user asks to "set up a workflow", "make a new workflow for X", "scaffold the three files", "start a new Claude project", "set up a folder for Claude", "new project for Cypress / PR-review / API-testing / code-review", or any phrasing that implies creating a dedicated folder where Claude should inherit identity and project context. Supports both walkthrough mode (one question at a time) and quick-scaffold mode (when the opening message already has enough detail). Auto-detects whether the target is inside an Obsidian vault and adds vault-friendly frontmatter only then; for plain code projects, emits the lesson templates verbatim. Does NOT execute the workflow — it only creates it.
user-invocable: true
argument-hint: "[workflow name, e.g. 'cypress-test-creation']"
---

# workflow-setup

Scaffold a dedicated folder with three files — `CLAUDE.md`, `CONTEXT.md`, `REFERENCES.md` — at the start of any new project, so Claude inherits identity and project context automatically whenever it runs inside that folder. Works for code repos, Obsidian vaults, content projects, or anywhere else — the target folder is whatever the user tells you.

## What the three files do

| File | Purpose |
|---|---|
| `CLAUDE.md` | Who Claude is and the house rules — read first by Claude Code at session start. Durable across every conversation inside this folder. |
| `CONTEXT.md` | What the current project is, what good looks like, what to avoid. Project-specific and refreshed as the project evolves. |
| `REFERENCES.md` | Background material Claude should know about but doesn't need to act on directly — examples, links, notes. |

The payoff: once the files exist, every conversation in that folder starts with Claude already knowing who it's working for and what the project is. You stop re-explaining context every time.

## Flow

### Step 1 — Triage the input

Two entry paths:

- **Quick-scaffold mode** — the user's opening message already contains enough detail (workflow name + topic + at least a rough sense of what they're building). Infer the rest yourself, confirm in one sentence, and proceed to Step 3 (write files). Do NOT pepper them with questions when the answers are already on the table.
- **Walkthrough mode** — user gave just a name, or nothing. Ask one question at a time (Step 2).

Heuristic: if you can write all three files without blocking, do it. If you'd have to invent the project description, ask.

### Step 2 — Collect the information (walkthrough mode)

Ask in this order, one at a time:

**Q1 — Workflow name.**
> What should this workflow be called? A short kebab-case name works well — e.g. `cypress-test-creation`, `pr-review`, `api-testing`, `frontend-playwright`.

Convert whatever they give you to kebab-case for the folder name. Preserve their original phrasing for use inside the files.

**Q2 — Target location.**
> Where should the folder live? A few common buckets:
> 1. **Code project** — `~/code/<name>/` or wherever you keep repos
> 2. **Obsidian vault** — e.g. `~/obsidian/<vault-name>/<name>/`
> 3. **Custom path** — paste a full path.

If the user names a location casually ("my vault", "~/code", "the QA vault"), map it to the full path. If unsure, show the resolved path and confirm before writing.

**Detect the environment**: check whether the final path is inside an Obsidian vault. Primary heuristic — walk up from the target until you find a `.obsidian/` directory (the deterministic signal). Fallback heuristic — path starts with `~/obsidian/` or `~/Library/Mobile Documents/iCloud~md~obsidian/` (common vault locations). Store the result as `IS_VAULT` (true/false). This determines which template variant you use in Step 4.

**Q3 — Identity (for CLAUDE.md).**
> Who is Claude working for in this folder, and what's the role? One sentence.
> Examples: *"Helping Brandon with Cypress E2E test authoring for a staging e-commerce app."* · *"Helping Brandon review pull requests in a React monorepo."* · *"Helping Brandon scope API test coverage for a Node/Express backend."*

**Q4 — Project (for CONTEXT.md).**
Three short sub-questions, presented together so the user can answer in one message:
> Tell me three things about the project:
> 1. **What are you building?** (2–3 sentences)
> 2. **What does a good result look like?** (what would you paste into a review that would make you nod?)
> 3. **What should Claude avoid?** (common mistakes, banned patterns, tone you don't want)

**Q5 — References (for REFERENCES.md).**
> Any reference material? You can skip this and leave the file with placeholders, or give me any of:
> - A link or two Claude should know about (docs, staging URL, design doc)
> - A quick example of output you liked (paste a snippet)
> - Notes — house conventions, gotchas, anything Claude should know but shouldn't act on

If the user skips, write REFERENCES.md with the placeholder template so they can fill it later.

### Step 3 — Pre-flight check

Before writing anything:

1. **Verify parent directory exists** (use Glob or Bash `ls`).
2. **Check the target workflow folder does NOT already exist.** If it does, STOP and ask:
   > A folder already exists at `<path>`. Want me to: (a) add the three files alongside existing contents, (b) pick a different name, or (c) bail?
   Do not silently overwrite.
3. **Confirm the resolved path** back to the user in one line: *"Scaffolding at `<full/path>` — confirm?"* If quick-scaffold mode and the path is unambiguous (e.g. the user gave a full path), skip the confirm and just write.

### Step 4 — Write the three files

Use the templates below. Every filled-in `<placeholder>` should be replaced with the user's actual content — no leftover angle brackets or square brackets in the final files. Keep section headings VERBATIM across both variants (they match the teaching material).

**Two variants — pick based on the `IS_VAULT` flag from Step 2:**

- **`IS_VAULT = true`** → prepend the frontmatter block shown first. Makes the workflow queryable from Obsidian Bases + tag search.
- **`IS_VAULT = false`** (code project, content folder, anything outside a vault) → omit frontmatter entirely. Ship the plain-markdown body exactly as the lesson teaches.

#### `CLAUDE.md`

Frontmatter (vault only):
```yaml
---
type: workflow-identity
tags:
  - workflow
  - claude
workflow: <kebab-case-name>
created: <YYYY-MM-DD>
---
```

Body (always):
```markdown
# Identity

You are helping <NAME> with <ROLE / WHAT THEY DO IN THIS WORKFLOW>.

## Rules
- Write in plain, clear language
- Ask clarifying questions before making assumptions
- When you are unsure, say so
<any extra rules the user gave during Q3>
```

If the user didn't supply a name, default `<NAME>` to `Brandon`. If they want a different name, they'll have already told you during Q3.

#### `CONTEXT.md`

Frontmatter (vault only):
```yaml
---
type: workflow-context
tags:
  - workflow
  - context
workflow: <kebab-case-name>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

Body (always):
```markdown
# Current Project

## What we are building
<2–3 sentences from Q4.1>

## What good looks like
<user's answer to Q4.2>

## What to avoid
<user's answer to Q4.3 — bullet list if multiple items>
```

#### `REFERENCES.md`

Frontmatter (vault only):
```yaml
---
type: workflow-references
tags:
  - workflow
  - references
workflow: <kebab-case-name>
created: <YYYY-MM-DD>
---
```

Body (always):
```markdown
# References

## Examples of good work
<user's example from Q5, or placeholder if skipped: "[Paste an example or describe what you liked about it]">

## Relevant links
<bulleted list of links, or placeholder: "[URLs, docs, resources for this project]">

## Notes
<user's notes, or placeholder: "[Anything else Claude should know]">
```

Write the files using the Write tool. Do NOT run shell commands (`mkdir`, `touch`, `cat <<EOF`) to create them — use the Write tool for each file so the harness tracks state properly.

If the target directory doesn't exist yet, create it with a single Bash `mkdir -p <path>` before the Write calls.

### Step 5 — Confirm + next steps

After writing, show the user a tree view of what was created:

```
<workflow-name>/
├── CLAUDE.md          (Identity + house rules)
├── CONTEXT.md         (Current project)
└── REFERENCES.md      (Background material)
```

Then tell them how to use it:

> **Next step:** `cd <full/path>` and run `claude`. The three files will be loaded automatically at session start. Ask Claude something about the project and notice how much more specific the answer is than a cold start.

If `IS_VAULT` was true, add a second line:
> To edit later, open the vault in Obsidian — the files are tagged `#workflow` so they'll show up in any Base filtering on that tag.

If `IS_VAULT` was false, add instead:
> To edit later, open the files in your editor of choice. They're plain markdown — no tooling required.

If the workflow is for something that would benefit from a companion QA-toolkit skill (e.g. they said "cypress test creation" → mention the `/cypress-test-creation` skill; "api testing" → `/api-testing`; "pr review" → `/pr-research-test-plan`), note the relevant one in a single closing line. Do not list skills that aren't clearly relevant.

## Edge cases

- **User gives only the workflow name** (e.g. just `/workflow-setup cypress`). Enter walkthrough mode starting at Q2. Don't skip ahead.
- **User pastes a huge project description into the opening message.** Infer everything, confirm the inferred values in one short block (*"Name: `cypress-test-creation` · Path: `~/code/cypress-test-creation/` · Building: …"*), then write. Saves them re-typing.
- **Target folder already has content** → ask before writing (Step 3 case).
- **User wants to update an existing workflow instead of creating a new one** → out of scope for this skill. Tell them: *"This skill scaffolds new workflows. To update an existing one, open the file in Obsidian and edit directly — or ask me to edit `<path>/CONTEXT.md` as a plain file edit."*
- **User wants nested sub-workflows** (the "Section 3" multi-folder pattern with `script-lab/`, `production/`, `distribution/`, each with their own `CONTEXT.md`). Out of scope. Tell them: *"That's the nested pattern — create the parent workflow with this skill, then run the skill again inside each subfolder for per-sub-workflow CONTEXT.md files. The parent CLAUDE.md at the root applies to every subfolder automatically."*
- **User wants to skip a file entirely** (e.g. "I don't need REFERENCES.md yet"). Scaffold it anyway with placeholders — an empty REFERENCES.md costs nothing and prevents a re-run later. Mention it: *"REFERENCES.md created with placeholders — fill when you have material."*
- **Path contains spaces** (e.g. the user points at `~/obsidian/<vault>/PR Review/`). Accept it; quote paths in any shell commands. Obsidian handles folder names with spaces fine.

## What you do NOT do

- **Do not run `claude` or open a Claude session inside the workflow.** The skill scaffolds; the user drives.
- **Do not invent project details.** If the user didn't supply a good-looks-like statement, leave the placeholder — don't hallucinate one.
- **Do not modify files outside the target folder.** No edits to the vault's root `CLAUDE.md`, no sidebar/property changes, no `Welcome.md` updates unless explicitly asked.
- **Do not scaffold the Section 3 nested architecture** (script-lab / production / distribution sub-workflows). That's a richer pattern; it deserves a separate skill if/when the user wants it.
- **Do not commit or push.** Creating the files is the whole job. Any git work is a follow-up the user can request.
