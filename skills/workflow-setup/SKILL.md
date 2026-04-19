---
name: workflow-setup
description: Scaffold a Claude workflow at the start of any new project — a code repo, an Obsidian folder, a consulting practice, a content project, anywhere Claude will be working repeatedly. Two shapes — **single-folder** (one `CLAUDE.md` + one `CONTEXT.md` + optional `REFERENCES.md`, for focused projects or incremental starts) and **multi-workspace** (parent `CLAUDE.md` with a routing table + 2–3 sub-workspace folders each with their own `CONTEXT.md`, for projects with distinct modes of work like `script-lab/production/distribution` or `planning/src/docs`). Use when the user asks to "set up a workflow", "make a new workflow for X", "scaffold a project with sub-workspaces", "scaffold the three files", "start a new Claude project", "set up a folder for Claude", "new project for Cypress / PR-review / API-testing / code-review", or any phrasing that implies creating a dedicated folder where Claude should inherit identity and project context. Supports both walkthrough mode (one question at a time) and quick-scaffold mode (when the opening message already has enough detail). Auto-detects whether the target is inside an Obsidian vault and adds vault-friendly frontmatter only then. Does NOT execute the workflow — it only creates it.
user-invocable: true
argument-hint: "[workflow name, e.g. 'cypress-test-creation']"
---

# workflow-setup

Scaffold a Claude workflow so every future conversation inside the folder starts with Claude already knowing who it's working for and what the project is. Two shapes are supported:

- **Single-folder** — one flat folder with `CLAUDE.md` + `CONTEXT.md` + optional `REFERENCES.md`. Good for focused projects (a single skill, a single test suite, a single research project) or as an incremental start that can grow into the multi-workspace shape later.
- **Multi-workspace** — parent folder with a routing `CLAUDE.md` + 2–3 sub-workspace folders (e.g. `script-lab/`, `production/`, `distribution/`), each with its own `CONTEXT.md`. This is the canonical three-layer pattern for projects that have distinct modes of work.

Default to single-folder unless the user clearly has multiple modes of work. You can always add workspaces later.

## What the files do

| File | Purpose |
|---|---|
| `CLAUDE.md` | **Map.** Routing file at the top level: project identity, list of workspaces, a routing table ("for task X, read CONTEXT.md in folder Y"), and naming conventions. Fits on one screen. Read first by Claude Code at session start. |
| `CONTEXT.md` | **Room.** One per workspace. Describes the *work* — what's being built, who it's for, what good looks like, what to avoid. Refreshed as the project evolves. |
| `REFERENCES.md` | **Additive companion** (not in the core three-layer lesson, but useful in practice). Background material Claude should know about but doesn't need to act on — links, example outputs, notes. Skip it if you don't have material yet. |

Skills / tools plug into specific workspaces via the routing table. This skill does not scaffold the skills layer — add skill references to `CLAUDE.md` manually when you need them.

## Flow

### Step 1 — Triage the input

Two entry paths:

- **Quick-scaffold mode** — the user's opening message already contains enough detail (workflow name + what they're building + enough signal to tell single vs multi-workspace apart). Infer the rest yourself, confirm in one sentence, and proceed to Step 3 (write files). Do NOT pepper them with questions when the answers are already on the table.
- **Walkthrough mode** — user gave just a name, or nothing. Ask one question at a time (Step 2).

Heuristic: if you can write every file without blocking, do it. If you'd have to invent work-level details, ask.

### Step 2 — Collect the information (walkthrough mode)

Ask in this order, one at a time. Aim for a complete scaffold in 5–6 short questions.

**Q1 — Workflow name.**
> What should this workflow be called? A short kebab-case name works well — e.g. `cypress-test-creation`, `pr-review`, `api-testing`, `my-consulting-practice`.

Convert whatever they give you to kebab-case for the folder name. Preserve their original phrasing for use inside the files.

**Q2 — Target location.**
> Where should the folder live? A few common buckets:
> 1. **Code project** — `~/code/<name>/` or wherever you keep repos
> 2. **Obsidian vault** — e.g. `~/obsidian/<vault-name>/<name>/`
> 3. **Custom path** — paste a full path.

If the user names a location casually ("my vault", "~/code", "the QA vault"), map it to the full path. If unsure, show the resolved path and confirm before writing.

**Detect the environment**: check whether the final path is inside an Obsidian vault. Primary heuristic — walk up from the target until you find a `.obsidian/` directory (the deterministic signal). Fallback heuristic — path starts with `~/obsidian/` or `~/Library/Mobile Documents/iCloud~md~obsidian/` (common vault locations). Store the result as `IS_VAULT` (true/false). This determines which template variant you use in Step 4.

**Q3 — Shape.**
> Do you want this as a **single-folder** workflow (one `CONTEXT.md`, one focused mode of work) or a **multi-workspace** workflow (2–3 sub-workspaces for different modes of work, each with their own `CONTEXT.md`)?
>
> Default is single — you can always add workspaces later when you know you need them. Pick multi-workspace if you already know the project has distinct modes, e.g. content creator (`script-lab/production/distribution`), consultant (`client-a/client-b/templates/business-dev`), or developer (`planning/src/docs/ops`).

Store the result as `SHAPE` (`single` or `multi`). If `multi`, ask Q3a:

**Q3a — Workspace names** (multi only).
> List 2–4 workspaces and a one-line purpose for each. Examples:
> - `script-lab` — Idea development, writing, drafts
> - `production` — Building and producing content
> - `distribution` — Publishing, scheduling, repurposing
>
> Keep it to **2–3 to start**. Four is the ceiling. Adding more later is trivial; pruning is painful.

If the user proposes more than 4, push back once: *"That's a lot for the scaffold — common trap is too many workspaces (the architecture overhead overtakes the work). Want to start with 2–3 and add more later?"* Accept their call if they insist.

**Q4 — Identity (for `CLAUDE.md`).**
> Who is Claude working for in this folder, and what's the role? One sentence.
> Examples: *"Helping Brandon with Cypress E2E test authoring for a staging e-commerce app."* · *"Helping Brandon review pull requests in a React monorepo."* · *"Helping Brandon scope API test coverage for a Node/Express backend."*

**Q5 — Project content.**

If `SHAPE = single`:
> Tell me three things about the project:
> 1. **What are you building?** (2–3 sentences)
> 2. **What does a good result look like?** (what would you paste into a review that would make you nod?)
> 3. **What should Claude avoid?** (common mistakes, banned patterns)

If `SHAPE = multi`:
> For each workspace, tell me the same three things — what's built there, what good looks like, what to avoid. You can batch them in one message (e.g. *"script-lab: writing scripts for YouTube explainers on AI tooling; good = 8-min, 1 hook + 3 beats + 1 takeaway; avoid clickbait titles"*). Keep each under a short paragraph.

**Q6 — Naming conventions (optional).**
> Any file naming conventions Claude should follow in this folder? Examples: *"Drafts: `topic-name_draft.md`"*, *"Specs: `feature-name_spec.md`"*, *"Decision records: `YYYY-MM-DD-decision-title.md`"*. Skip if you don't have any yet.

**Q7 — References (optional).**
> Any reference material for `REFERENCES.md`? Links, an example of output you liked, notes Claude should know about but shouldn't act on. Skip and I'll leave placeholders.

If the user skips Q6 or Q7, omit the section (Q6) or write placeholders (Q7).

### Step 3 — Pre-flight check

Before writing anything:

1. **Verify parent directory exists** (use Glob or Bash `ls`).
2. **Check the target workflow folder does NOT already exist.** If it does, STOP and ask:
   > A folder already exists at `<path>`. Want me to: (a) add the files alongside existing contents, (b) pick a different name, or (c) bail?
   Do not silently overwrite.
3. **Confirm the resolved path** back to the user in one line: *"Scaffolding at `<full/path>` — confirm?"* If quick-scaffold mode and the path is unambiguous (e.g. the user gave a full path), skip the confirm and just write.

### Step 4 — Write the files

Pick template variants from the sections below based on `SHAPE` and `IS_VAULT`.

**Two axes — four combinations:**

| `SHAPE` | `IS_VAULT` | Files to write |
|---|---|---|
| single | false | `CLAUDE.md` (plain, single-folder body) · `CONTEXT.md` (plain) · `REFERENCES.md` (plain) |
| single | true | same, with vault frontmatter prepended to each |
| multi | false | parent `CLAUDE.md` (plain, multi-workspace body with routing table) · for each sub-workspace: `<workspace>/CONTEXT.md` (plain) · optionally `REFERENCES.md` at the parent root |
| multi | true | same, with vault frontmatter prepended to each |

Vault frontmatter (prepend to every scaffolded file when `IS_VAULT = true`):
```yaml
---
type: workflow-<identity|context|references>
tags:
  - workflow
  - <claude|context|references>
workflow: <kebab-case-name>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>   # CONTEXT.md only
---
```

Write files with the Write tool — one call per file. Do NOT use shell heredocs (`cat <<EOF`). If the target directory doesn't exist yet, create it with a single Bash `mkdir -p <path>` (including sub-workspace subdirs) before the Write calls.

#### `CLAUDE.md` — single-folder body

```markdown
# <Project Name>

<1–2 sentence description from Q4 — what the project is, who Claude is helping, what the role is>

## Naming conventions
<from Q6, bullet list — omit this section entirely if user skipped Q6>

## Rules
- <1–3 project-specific rules max, from any extras the user gave in Q4. Omit entirely if none.>
```

**Hard constraints on `CLAUDE.md`:**
- Fit on one screen (< 50 lines).
- No AI-personality rules ("be clear", "ask questions", "say when unsure"). These belong nowhere in this file — they're the anti-pattern the lesson calls out as Mistake 4. Only write rules that are *specific to this project* and would surprise someone who didn't know the project.

#### `CLAUDE.md` — multi-workspace body

```markdown
# <Project Name>

<1–2 sentence description from Q4>

## Workspaces
- /<workspace-a> — <one-line purpose from Q3a>
- /<workspace-b> — <one-line purpose from Q3a>
- /<workspace-c> — <one-line purpose from Q3a>

## Routing
| Task | Go to | Read |
|------|-------|------|
| <task type from Q3a's workspace-a purpose> | /<workspace-a> | CONTEXT.md |
| <task type from workspace-b purpose> | /<workspace-b> | CONTEXT.md |
| <task type from workspace-c purpose> | /<workspace-c> | CONTEXT.md |

## Naming conventions
<from Q6, bullet list — omit section if user skipped Q6>

## Rules
- <1–3 project-specific rules max. Omit entirely if none.>
```

Same one-screen / no-personality constraint applies.

#### `CONTEXT.md` — body (same for single and multi; write one per workspace in multi mode)

```markdown
# <Workspace Name> — Context

> Last updated: <YYYY-MM-DD>

## What we are building
<2–3 sentences from Q5.1 — describe the work, the audience, the constraint>

## What good looks like
<from Q5.2 — concrete signals of success>

## What to avoid
<from Q5.3 — common mistakes, banned patterns; bullet list if multiple items>
```

**Hard constraints on `CONTEXT.md`:**
- Describe the work, not the AI. If the user's Q5 answer is dominated by AI-personality instructions ("be concise", "be creative", "think step by step"), push back once: *"Those sound like personality instructions — the file works better if it describes the audience, success criteria, and anti-patterns of the work itself. Want to rephrase?"* Accept their wording if they insist.
- Keep under one page. If a workspace's context genuinely exceeds one page, that's often a signal it should be two workspaces.

#### `REFERENCES.md` — body (same for both shapes; single file at root)

```markdown
# References

## Examples of good work
<from Q7, or placeholder: "[Paste an example or describe what you liked about it]">

## Relevant links
<bulleted list of links, or placeholder: "[URLs, docs, resources for this project]">

## Notes
<from Q7, or placeholder: "[Anything else Claude should know]">
```

### Step 5 — Confirm + next steps

After writing, show the user a tree view of what was created. For single-folder:

```
<workflow-name>/
├── CLAUDE.md          (Project identity + routing)
├── CONTEXT.md         (Current project)
└── REFERENCES.md      (Background material)
```

For multi-workspace:

```
<workflow-name>/
├── CLAUDE.md          (Project identity + routing table)
├── REFERENCES.md      (Background material — optional)
├── <workspace-a>/
│   └── CONTEXT.md
├── <workspace-b>/
│   └── CONTEXT.md
└── <workspace-c>/
    └── CONTEXT.md
```

Then tell them how to use it:

> **Next step:** `cd <full/path>` and run `claude`. The `CLAUDE.md` is loaded automatically at session start, and the routing table points Claude into the right workspace's `CONTEXT.md` based on the task.

If `IS_VAULT` was true, add:
> To edit later, open the vault in Obsidian — the files are tagged `#workflow` so they'll show up in any Base filtering on that tag.

If `IS_VAULT` was false, add instead:
> To edit later, open the files in your editor of choice. They're plain markdown — no tooling required.

If the workflow is for something that would benefit from a companion QA-toolkit skill (e.g. they said "cypress test creation" → mention `/cypress-test-creation`; "api testing" → `/api-testing`; "pr review" → `/pr-research-test-plan`), note the relevant one in a single closing line. Do not list skills that aren't clearly relevant.

Close with one habit nudge: *"`CONTEXT.md` is a working document — bump the `Last updated` line when the project changes. Stale context is the #1 reason Claude output drifts."*

## Edge cases

- **User gives only the workflow name** (e.g. just `/workflow-setup cypress`). Enter walkthrough mode starting at Q2. Don't skip ahead.
- **User pastes a huge project description into the opening message.** Infer everything, confirm the inferred values in one short block (*"Name: `cypress-test-creation` · Path: `~/code/cypress-test-creation/` · Shape: single · Building: …"*), then write. Saves them re-typing.
- **Target folder already has content** → ask before writing (Step 3 case).
- **User proposes 5+ workspaces.** Push back once (see Q3a). If they insist, proceed — but scaffold only what they asked for; don't silently trim.
- **User wants a skills/tools layer wired in** (Layer 3 from the lesson). Out of scope for this skill — it only scaffolds the map and the rooms. Tell them: *"Add skill references to your `CLAUDE.md` manually — one column in the routing table, one line per skill. The skill loader picks them up automatically."*
- **User wants to update an existing workflow instead of creating a new one** → out of scope for this skill. Tell them: *"This skill scaffolds new workflows. To update an existing one, open the file in Obsidian or your editor and edit directly — or ask me to edit `<path>/CONTEXT.md` as a plain file edit."*
- **User wants to skip a file entirely** (e.g. "I don't need REFERENCES.md yet"). For `REFERENCES.md`, scaffold it with placeholders — an empty file costs nothing and prevents a re-run. For `CONTEXT.md`, refuse — it's load-bearing.
- **Path contains spaces** (e.g. `~/obsidian/<vault>/PR Review/`). Accept it; quote paths in any shell commands. Obsidian and macOS both handle folder names with spaces fine.
- **User asks for per-client or per-sub-project templates** (consultant pattern from the lesson's Example 2 — `client-alpha/`, `client-beta/`, each with their own `CONTEXT.md`). That *is* multi-workspace mode — each client is a workspace. Proceed normally.

## What you do NOT do

- **Do not run `claude` or open a Claude session inside the workflow.** The skill scaffolds; the user drives.
- **Do not invent project details.** If the user didn't supply a good-looks-like statement, leave the placeholder — don't hallucinate one.
- **Do not write AI-personality rules** ("be concise", "ask clarifying questions", "think step by step") into `CLAUDE.md` or `CONTEXT.md`. Describe the *work*, not how Claude should act. Personality instructions are Mistake 4 — they crowd out context that actually shapes output.
- **Do not let `CLAUDE.md` grow past ~50 lines.** If the content doesn't fit, push detail down into a workspace `CONTEXT.md`. A long `CLAUDE.md` is Mistake 1 — Claude burns tokens on boilerplate and the routing table gets buried.
- **Do not scaffold more than 4 workspaces.** If the user asks for more, push back once (too many workspaces is Mistake 3). Proceed if they insist.
- **Do not modify files outside the target folder.** No edits to a vault's root `CLAUDE.md`, no sidebar/property changes, no `Welcome.md` updates unless explicitly asked.
- **Do not scaffold a skills/tools layer.** Users can add skill references to `CLAUDE.md` manually.
- **Do not commit or push.** Creating the files is the whole job. Any git work is a follow-up the user can request.
