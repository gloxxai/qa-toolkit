---
name: prompt-maker
description: Guide the user through composing a well-structured prompt using the Five-Part Framework (Identity, Task, Context, Constraints, Output Format). Use this whenever the user asks for help writing, building, structuring, drafting, or shaping a prompt — including phrasings like "help me write a prompt", "make me a prompt for X", "build a prompt that will X", "walk me through writing a prompt", "I need to structure a prompt", "prompt-maker", or any intent to compose a prompt that needs structure (even if they don't use the word "prompt" — e.g. "I want to describe this task to Claude carefully"). A key feature: when gathering Context, the user can either type a short description or point the skill at a file or folder path, and the skill will read and embed those files as structured context. Do not trigger for direct one-shot tasks where the user is asking Claude to execute something ("write me a blog post", "summarize this file", "debug this function") — trigger only when the meta-task is composing the prompt itself.
user-invocable: true
argument-hint: "[what the prompt should accomplish]"
---

# prompt-maker

Walk the user through the Five-Part Prompt Framework — Identity, Task, Context, Constraints, Output Format — and assemble a paste-ready prompt at the end.

## Why this framework

A prompt is an instruction set. When output disappoints, the fix is almost always a missing part, not weaker phrasing. This skill makes sure no part is accidentally skipped. It's also a diagnostic tool — after using a prompt, if the answer falls short, the user can walk the five parts and ask *which one did I leave out?*

You do not need to use all five parts for every task. The right set depends on complexity:

| Task type | Parts to include |
| --- | --- |
| Simple (rename, fix typo, one-liner) | Task only |
| Creative (write, draft, design) | Identity + Task + Constraints + Output Format |
| Complex (build, analyze, review multi-step work) | All five |
| Ongoing project (reused across many conversations) | Identity + Context belong in a `CLAUDE.md` file; each prompt carries Task + Constraints |

## Flow

### Step 1 — Greeting and triage

Open with a brief greeting that surfaces the decision about which parts are needed. Something like:

> Happy to help you build a solid prompt. Quick orientation: is this a **simple** ask (rename, quick fix), a **creative** task (draft, write, design), a **complex** task (analyze, review, multi-step), or an **ongoing project** you'll reuse this prompt for? If you're not sure, just say *ask me all five* and I'll walk you through every part.

Branch on their answer:

- **Simple** → collect only Task, then assemble.
- **Creative** → collect Identity, Task, Constraints, Output Format. Skip Context.
- **Complex** / *ask me all five* / unsure → collect all five.
- **Ongoing** → see "Ongoing-project branch" below.

If the user's opening message already contains a rich description of what they want (e.g. a paragraph describing the task), infer the task type yourself, confirm it in one sentence, and move straight to collecting the missing parts.

### Step 2 — Walk the parts

Ask one question at a time unless the user asks for batch mode (e.g. "just give me all the questions at once"). After each answer, briefly confirm ("Got it — X.") and move on. If the user seems stuck, offer 2-3 concrete examples tuned to what they've said so far.

#### Identity
> Who should Claude be right now? (Role, technical depth, perspective.)
> Examples: *senior Cypress engineer* · *technical copywriter* · *Solidity auditor* · *product strategist* · *staff data scientist*.

If the user says their project already has a `CLAUDE.md` that defines the role, skip Identity and note that it's inherited.

#### Task
> What specifically do you need Claude to do? Use a clear action verb and a defined scope.
> Examples: *"Write 5 Cypress specs for the login flow."* · *"Audit this `upgradeTo()` call for storage collisions."* · *"Summarize this article into 3 bullet takeaways."*

A good signal that Task is well-scoped: a stranger could start on it without asking five follow-up questions.

#### Context — the flexible one

Ask it as a choice so the user knows both options:

> For context, you have two options:
>
> 1. **Type it** — give me a sentence or two of background, specs, audience, or any info Claude would need.
> 2. **Point at files** — give me a file path or folder path and I'll read those files and embed them as context.
>    (e.g. `/path/to/your/notes/` to load a whole folder of notes, or a single file path for one file.)
>
> You can also combine them — a sentence of framing *plus* a path.

**Handling a path:**

1. Verify the path exists. Use the Read tool for a single file, or the Glob tool for a folder. If the path doesn't exist, say so plainly: "I couldn't find that path — can you double-check, or give me a text description instead?"
2. If it's a single file, Read it and embed the content under a `--- Loaded from <path> ---` header.
3. If it's a folder, Glob with pattern `**/*.md` by default. If the user says "everything" or mentions non-markdown files specifically, use `**/*`. Read each matched file.
4. **Size cap — roughly 50 KB of embedded content total** (≈ 12,500 tokens). If the target exceeds this, do not silently truncate. Offer the user three choices:
   > That target is bigger than what fits cleanly in one prompt. Want me to (a) narrow to a subfolder, (b) keep only the top N files most relevant to the task, or (c) summarize each file first?

**Handling text:** use it verbatim in the Context slot.

**Handling both:** concatenate — the user's text framing goes first, then the loaded file contents.

#### Constraints
> What should Claude *not* do? List 1-5 things — limits, boundaries, negative instructions.
> Examples: *Do not use jargon.* · *Don't suggest paid tools.* · *Keep under 300 words.* · *No inline comments.* · *Don't start with "In today's world".*

Every constraint the user sets is a mistake Claude won't make twice. A useful prompt to the user if they're stuck: *think about what's annoyed you in past AI outputs — those annoyances are your constraints.*

#### Output Format
> What shape should the answer take? (List, table, code block, markdown doc, JSON, a draft with `[PLACEHOLDER]` sections, etc.)
> Examples: *"A 3-bullet summary."* · *"A Cypress spec file with `describe`/`it` structure."* · *"A markdown table with columns Severity | Finding | Recommendation."*

### Step 3 — Assemble the final prompt

Once you have all the needed parts, assemble them in a fenced code block. Label each part inline with square brackets. The label style should match:

```
[IDENTITY] You are a senior Cypress engineer who writes E2E specs for web apps.

[TASK] Write a Cypress spec covering the guest checkout flow — happy path plus two failure modes (expired card, address-validation failure).

[CONTEXT] React SPA. Staging URL: https://staging.example.com. All interactive elements have `data-cy="..."` attributes. Custom commands available: `cy.loginAs()`, `cy.seedCart()`. Payment step uses Stripe — intercept `POST /api/charge` and mock the response.

[CONSTRAINTS] Use `cy.data(...)` selectors only — no CSS or text selectors. No `cy.wait(ms)` — use `cy.intercept()` aliases. Reuse `cy.loginAs('guest')`. Keep the spec under 200 lines.

[OUTPUT] One Cypress spec file. `describe('Checkout flow')` with three `it(...)` blocks. Include intercept aliases `@charge` and `@validateAddress`. End with a one-paragraph comment listing scenarios NOT covered.
```

When Context was loaded from files, embed them under the [CONTEXT] label like this:

```
[CONTEXT]
Project framing: <user's text, if any>

--- Loaded from /path/to/notes/Five-Part Prompt Framework.md ---
<full file contents>

--- Loaded from /path/to/notes/Prompt Chunking.md ---
<full file contents>
```

Omit parts the user skipped. Don't emit `[IDENTITY] (skipped)` — just leave the label off entirely.

After the code block, close with a single line:

> *Paste this into Claude. If the output disappoints, walk the five parts and ask which one you skipped.*

## Ongoing-project branch

When the user says this is an ongoing project, the five parts split across two places:

- **Identity + Context** → belong in a `CLAUDE.md` file at the project root, so every future prompt inherits them automatically.
- **Task + Constraints + Output Format** → go in each prompt, per task.

Before walking the parts, ask:

> For ongoing projects, Identity and Context usually live in a `CLAUDE.md` file at your project root — that way every future prompt inherits them without you re-typing. Two options:
>
> 1. Want me to draft a `CLAUDE.md` for you, plus a shorter per-task prompt template?
> 2. Or include everything inline in a single prompt for now, and you can extract later?

If they choose (1), produce two artifacts: a `CLAUDE.md` draft (Identity + Context sections) and a task-level prompt template (Task, Constraints, Output Format placeholders). If (2), just walk all five as normal.

## Edge cases

- **Batch mode**: if the user wants all questions at once, present them as a numbered list in one message. Assemble after they reply.
- **Vague answer on Identity**: offer 3 concrete options based on the Task verb — code tasks → "senior engineer", docs → "technical writer", analysis → "analyst in the relevant domain". If the user says "pick for me", do so.
- **Context path not found**: say so plainly. Offer: retry with a corrected path, or switch to text description.
- **Context exceeds 50 KB**: present the three narrowing options. Don't silently truncate — silent truncation hides information from the user that they'd want to know about.
- **User abandons mid-flow**: no cleanup needed. The conversation is the state.
- **User wants to change an earlier answer**: accept it, update, keep going.

## What you do NOT do

- **Do not execute the prompt.** This skill composes a prompt; the user runs it in the conversation of their choice. Don't answer the task itself.
- **Do not enforce all five parts when the task is simple.** The triage step exists so a one-line rename doesn't get wrapped in a five-section structure.
- **Do not silently truncate large file loads.** Always surface the size trade-off and let the user choose how to narrow.
- **Do not invent Context.** If the user doesn't supply it and doesn't point at a path, leave the slot empty — don't hallucinate project background.
